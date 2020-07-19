# New options to validate RSA signed JWT tokens

## Summary
> This post highlights value of the enhanced validate-jwt API Management policy, for use with RSA signed JWT tokens. It also provides samples which are missing from official documentation at the time of writing and related tips.

As part of the June release (https://azure.microsoft.com/en-us/updates/azure-api-management-update-june-2020/), two new configuration options for validate-jwt policy are available, to validate RSA signed JWT tokens. Prior to this, with the incumbent option (read below), API Management customers were required to provide Open ID Discovery endpoint themselves. The new options shift the burden away from customers, while also simplifying the composite SLA calculations for their solution.

I have come across this pain point, for customers that have custom OAUTH solutions to provide RSA signed JWT tokens, but do not have Open ID Connect Discovery endpoints. Typically these customers are along their cloud adoption journey, where they have not yet consolidated their Identity and Access Management solutions, onto platforms like Azure AD, OKTA, etc, that will provide global, highly available Open ID Discovery endpoints, in addition to token endpoints, out of the box. These new options are very useful in these scenarios.

## (Incumbent) Option - Open ID Discovery endpoint
In this option, policy requires an Open ID Discovery endpoint to be specified via openid-config element. The policy expects to browse this endpoint and JWK Urls in the response, to discover the modulus and exponent (n/e) pair, which it will use for validating incoming RSA signed JWT tokens.
 
```
<validate-jwt header-name="Authorization">
   <openid-config url="https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration" />
</validate-jwt>
```

## (New) Option - Certificate
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

## (New) Option - n/e pair
In this option, policy can directly reference the issuer's signing modulus and exponent (n/e) pair in-line via attributes. These are not secrets, so ok to have these inline and source controlled.

```
<!-- Assumption is that the certificate (rsa-cert.pfx), containing issuer's keys has already been uploaded to API Management -->

<validate-jwt header-name="Authorization">
  <issuer-signing-keys>
    <key e="AQAB" n="pO2+BpARfKknqyr8sDRG1VmLlw5gBv518n2/LVyh16lMg2YUmDAmJCRX0BJQl0LytR5FIK0PTFDWAofguReR4OiLzQjiwKt9yFrnyslSYy74Ux4AdvkqC2Rv+nJ7NYaLIF3e/+gYrdK0buxpqZq+Hlh6Qugc/pYnxgODT59UllNGbKWVW3tlLxyxhj2KwzWpruhh/ECS1dq487u0ZAwgNPJw05Ku0ZMt9PET35Z4kr295ugxZl/nfdnfqu3lFBtbIoegtc2Rsok8YW0q91PeHOWMSu1B6PK4oZnEe3lcHRvCQzijG1SvBAAAzRdNZ6lgHsXS9XWyk/FyMcUhqImwOPOAzRtSUoDTafuBiefmB1+Rm1ypUM3PJxOrCP3qfImTmlmPE5AFMzg+oZqyUlh/kodWrfvIn2BbQ2BLWywghcUvUV1U6ipqNKh2D9Tg0853hO1KBOKcX2BJCZkr5zjg7NaYLC4nl/nn8DVqcjxoNghsH+a6IzwoPkcz/yAB8KmsNmCO9Ru9bjgl7xMbuAWAWVQqC4QY7VFYTW8W7Inc0e97mU4CJ5zwPw2DiVpjQD0anAWY1L+k61FCi93NKYhIQLw44ijIXbCdLaXjKxXu8FEkFW3hi1duxEIgqHve5TfHTddWi2xjVfl8X8vH4rDJIniBlQYeNVe/PvTRqjTHr2E="></key>
  </issuer-signing-keys>
</validate-jwt>
```

## Tips
- Portal UI Glitch - At the time of writing, the Azure API Management portal (UI) does not seem to render the policy correctly when used with new options correctly. After saving policy, it seems to return to default/blank value, even though, under the hood the policy has been correctly applied. This could be a bug/feature yet to be released. Till then, to be sure what policy has taken effect, you can query the underlying ARM template via Azure Portal ('Settings --> Export Template' tab in API Management instance) or Resource Explorer. Alternatively, you can query the policy directly via Azure PowerShell module https://docs.microsoft.com/en-us/powershell/module/az.apimanagement/get-azapimanagementpolicy?view=azps-4.4.0
- Security Consideration - Using Option to reference certificate, requires customer to upload *.pfx certificate, which will cotain both public and private keys. To limit that exposure, a consideration should be made whether using n/e pair option is safer, as that is meant for public distribution.
- Tested Samples - please refer to following sample policies which I have tested out
- 
