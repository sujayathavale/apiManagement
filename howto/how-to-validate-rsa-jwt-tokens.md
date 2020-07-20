# Azure API Management - new options for validating RSA JWT tokens

## TL;DR
In this post, I highlight newly enhanced capabilities of [Validate JWT](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT) Azure API Management policy and the specific gap it addresses for customers. I also provide related recommendations, tips and policy samples, which are currently not yet available in Microsoft documentation.

## validate-jwt policy

The [Validate JWT](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT) Azure API Management policy has been around for a while, supporting validation of JWT tokens signed with either HS256 (symmetric) or RS256 (asymmetric) algorithms. I encourage use of RS256 algorithm over HS256, as it requires use of separate keys to sign (private key) and validate (public key) tokens. Only the validation key needs to be distributed to applications, like the API Management validate-jwt policy in question here, that want to validate these tokens. So far, with RS256, there was only one way to do this, using Open ID Discovery endpoint.


## Option - Open ID Discovery endpoint (existing)
In this option, policy requires an Open ID Discovery endpoint to be specified via openid-config element. API Management expects to browse this endpoint when evaluating policy, including JWK Urls in the response, to discover the modulus and exponent (n/e) pair, which it will use for validating incoming RS256 JWT tokens.
 
```
<validate-jwt header-name="Authorization">
   <openid-config url="https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration" />
</validate-jwt>
```
 
I think this is still the best option, as sourcing the keys from a remote endpoint like this, allows the keys to be rotated, without having to update validate-jwt policy.

__*Limitation/Gap*__

This option remains unavailable for customers, that either do not have an Open ID Discovery endpoint or have one that not usable. I have seen this for customers that have custom OAuth solutions, without Open ID Connect Discovery endpoints in place. Or if they have one, a lot of extra work needs to be done to make it resilient and provide API Management a line of sight to it. 

__*Long Term Solution*__
Most of my customers that are in this situation, are typically early on cloud adoption journey and have not yet consolidated their Identity and Access Management (IDAM) capabilities onto mature platforms like Azure AD, OKTA, etc., which provide global, highly available Open ID Discovery endpoints out of the box. Once they reach that point that is the long term solution, for use with this policy.

__*Intermediate Solution*__
The newly released configuration options, provide a way out in the interim, by allowing 2 additional ways to supply the keys to validate-jwt policy.

## Option - Certificate (new)
In this option, policy can reference a certificate uploaded to Azure API Management, via the certificate-id attribute. Refer
[upload a certificate](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-mutual-certificates#-upload-a-certificate)

```
<!-- Assumption is that the certificate (rsa-cert.pfx), containing issuer's keys has already been uploaded to API Management -->

<validate-jwt header-name="Authorization">
  <issuer-signing-keys>
    <key certificate-id="rsa-cert" />
  </issuer-signing-keys>
</validate-jwt>
```
__*Security Consideration*__

Azure API Management currently requires upload of *.pfx certificate, which will contain both public and private keys. To limit that exposure, a security consideration should be made, whether it is better to use the other 2 options instead.
 
## Option - n/e pair (new)
In this option, policy can directly reference the issuer's signing modulus and exponent (n/e) pair in-line via attributes. These are not secrets, so ok to have these inline and source controlled.

```
<!-- policy references JWT issuer's signing key modulus (n) and exponent (e) values in line -->

<validate-jwt header-name="Authorization">
  <issuer-signing-keys>
    <key e="AQAB" n="pO2+BpARfKknqyr8sDRG1VmLlw5gBv518n2/LVyh16lMg2YUmDAmJCRX0BJQl0LytR5FIK0PTFDWAofguReR4OiLzQjiwKt9yFrnyslSYy74Ux4AdvkqC2Rv+nJ7NYaLIF3e/+gYrdK0buxpqZq+Hlh6Qugc/pYnxgODT59UllNGbKWVW3tlLxyxhj2KwzWpruhh/ECS1dq487u0ZAwgNPJw05Ku0ZMt9PET35Z4kr295ugxZl/nfdnfqu3lFBtbIoegtc2Rsok8YW0q91PeHOWMSu1B6PK4oZnEe3lcHRvCQzijG1SvBAAAzRdNZ6lgHsXS9XWyk/FyMcUhqImwOPOAzRtSUoDTafuBiefmB1+Rm1ypUM3PJxOrCP3qfImTmlmPE5AFMzg+oZqyUlh/kodWrfvIn2BbQ2BLWywghcUvUV1U6ipqNKh2D9Tg0853hO1KBOKcX2BJCZkr5zjg7NaYLC4nl/nn8DVqcjxoNghsH+a6IzwoPkcz/yAB8KmsNmCO9Ru9bjgl7xMbuAWAWVQqC4QY7VFYTW8W7Inc0e97mU4CJ5zwPw2DiVpjQD0anAWY1L+k61FCi93NKYhIQLw44ijIXbCdLaXjKxXu8FEkFW3hi1duxEIgqHve5TfHTddWi2xjVfl8X8vH4rDJIniBlQYeNVe/PvTRqjTHr2E="></key>
  </issuer-signing-keys>
</validate-jwt>
```

# Azure Portal Tip
At the time of writing, the Azure API Management portal (UI) does not seem to render the policy correctly when used with new options. After saving policy, it seems to return to default/blank value, even though, under the hood the policy has been correctly applied. This is likely a bugfix/feature yet to be released. Till then, to be sure what policy has taken effect, you can query the underlying ARM template via Azure Portal ('Settings --> Export Template' tab in API Management instance) or Resource Explorer. Alternatively, you can query the policy directly via Azure PowerShell module [Get-AzApiManagementPolicy](https://docs.microsoft.com/en-us/powershell/module/az.apimanagement/get-azapimanagementpolicy?view=azps-4.4.0)


# Conclusion

As you have seen, use of Open ID Discovery endpoint, with Azure API Management validate-jwt policy, still remains the best and my recommended option for validating RSA JWT tokens. However, the newly released options provide a way forward for customers that are unable to do so, without imposing heavy technical and operational burden.

I hope you find the insights on scenarios, samples and tips I shared above useful.

If you like what you read, join our team as we seek to solve wicked problems within Complex Programs, Process Engineering, Integration, Cloud Platforms, DevOps & more!

Get In Touch


 

