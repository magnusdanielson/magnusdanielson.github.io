---
layout: post
title:  ".Net core for SharePoint onprem"
author: magnus
categories: [ SharePoint, app ]
image: assets/images/13.jpg
---
If you are still writing code for SharePoint onprem you have a few options to call SharePoint. Microsoft have provided
the CSOM-libraries on top of .Net 4.8. The excellent SharePoint community have built various versions of PnP, but that is also built on .Net 4.8. As they have stated on the github it is now archived (https://github.com/pnp/PnP-Sites-Core). The
SharePoint community is betting on .Net core and SharePoint online. That leaves us onprem developers with only one option.

## Build our own TokenHelper for .net core

Step 1. Register the SPTrustedSecurityTokenIssuer if you have not done that already.

```powershell
New-SelfSignedCertificate -CertStoreLocation cert:\localmachine\my -dnsname sp19b.dunite.local -FriendlyName "sp19Rsa256"
```

Then use the Certificate Managment tool to export the pfx and the cer files to the folder c:\certs
Finally register the SPTrustedSecurityTokenIssuer. Remeber to verify that the certificate uses Rsa256 algorithm

```powershell
$publicCertPath = "C:\certs\sp19brsa256.cer"
$certificate = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($publicCertPath)
New-SPTrustedRootAuthority -Name "SP19HighTrustRSA256" -Certificate $certificate
$realm = Get-SPAuthenticationRealm
$specificIssuerId = "a24341ca-8754-4781-8d90-3850263a50f0"
$fullIssuerIdentifier = $specificIssuerId + '@' + $realm
New-SPTrustedSecurityTokenIssuer -Name "High Trust SP19 RSA256" -Certificate $certificate -RegisteredIssuerName $fullIssuerIdentifier –IsTrustBroker
iisreset
```
I usually always setup the SharePoint machine to run SSL, but in this case it was not done yet, therefor we need:

```powershell
$serviceConfig = Get-SPSecurityTokenServiceConfig
$serviceConfig.AllowOAuthOverHttp = $true
$serviceConfig.Update()
```

You need to collect the following information, all values will differ on your farm.

```csharp
private const string CertificatPath = "C:\\certs\\sp19brsa256.pfx";
private string ClientId = "da64a40f-9946-4b38-8478-9c3aced2c96f";
private string IssuerId = "a24341ca-8754-4781-8d90-3850263a50f0";
private string KeyPassword = "your-password-to-the-pfx-certificate";
private string SharePointPrincipal = "00000003-0000-0ff1-ce00-000000000000";
private X509Certificate2? x5tCertificate;
private string x5tThumbPrint = string.Empty;
private string x5tThumb = string.Empty;
private string Realm = "305b6c1b-9bb5-4f48-b8b3-049578e91b1f";
```

Step 2. Open the certificate

```cs
char[] kpass = this.KeyPassword.ToCharArray();
using (FileStream ksfis = System.IO.File.OpenRead(CertificatPath))
{
    var bytes = new byte[ksfis.Length];
    ksfis.Read(bytes, 0, (int)ksfis.Length);
    X509Certificate2Collection collection = new X509Certificate2Collection();
    collection.Import(bytes, kpass, X509KeyStorageFlags.Exportable);
    foreach (X509Certificate2 cert in collection)
    {
        if (cert.HasPrivateKey)
        {
            this.x5tCertificate = cert;
            break;
        }
    }
    if (this.x5tCertificate != null)
    {
        this.x5tThumbPrint = Base64UrlEncoder.Encode(this.x5tCertificate.GetCertHash());
        this.x5tThumb = this.x5tCertificate.Thumbprint;
    }
}
```

Step 3. Create the app token 

```cs
long validFrom = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
long validTo = DateTimeOffset.UtcNow.AddHours(1).ToUnixTimeSeconds();
string issuer = $"{this.IssuerId}@{this.Realm}";
string nameid = $"{this.ClientId}@{this.Realm}";
string audience = $"{this.SharePointPrincipal}/{targetApplicationHostName}@{this.Realm}";
// CreateActorTokenJWT
var mytoken = JwtBuilder.Create()
          .WithAlgorithm(new RS256Algorithm(this.x5tCertificate))
          .AddClaim("nbf", validFrom)
          .AddClaim("exp", validTo)
          .AddClaim("aud", audience)
          .AddClaim("iss", issuer)
          .AddClaim("nameid", nameid)
          .AddClaim("trustedfordelegation", "true")
          .AddHeader("kid", this.x5tThumb)
          .AddHeader("x5t", this.x5tThumbPrint)
          .AddHeader("typ", "JWT")
          .Encode();

return mytoken;
```

Step 4. Request the data with REST, applying the Baerer-token

```cs
HttpClient client = new HttpClient();
client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", access_token);
client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
HttpResponseMessage response = await client.GetAsync(targetApplicationUri.Append(relativePath));
```

Step 5. Just incase you also need a user token

```cs
long nowMillis = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
long expMillis = DateTimeOffset.UtcNow.AddHours(1).ToUnixTimeSeconds();
string issuer = $"{this.IssuerId}@{this.Realm}";
string nameid = $"{this.ClientId}@{this.Realm}";
string audience = $"{this.SharePointPrincipal}/{targetApplicationHostName}@{this.Realm}";
string actorTokenString = this.CreateActorTokenJWT(issuer, audience, nameid, nowMillis, expMillis);
PrincipalContext ctx = new PrincipalContext(ContextType.Domain);
UserPrincipal user = UserPrincipal.FindByIdentity(ctx, accountName);
if (user != null)
{
    var mytoken = JwtBuilder.Create()
                     .WithAlgorithm(new NoneAlgorithm())
                      .AddClaim("nbf", nbf)
                      .AddClaim("exp", exp)
                      .AddClaim("aud", audience)
                      .AddClaim("iss", issuer)
                      .AddClaim("nameid", user.Sid.Value.ToLower())
                      .AddClaim("nii", "urn:office:idp:activedirectory")
                      .AddClaim("actortoken", actorToken)
                      .AddHeader("typ", "JWT")
                      .Encode();

    return mytoken;
}
throw new Exception("User not found");
```
## Summary
The same pattern can be applied with small modifications to get a user token too. We with tool
we can now easily call the REST Api in SharePoint onprem from .net core.