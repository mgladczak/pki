# LAB

## Offline Root CA

### Install ADCS Role for Offline Root CA

```powershell
Add-WindowsFeature Adcs-Cert-Authority -IncludeManagementTools
```

### Configure Offline Root CA

```powershell
Install-AdcsCertificationAuthority -CAType StandaloneRootCA -CACommonName "Bedrock Root Certificate Authority" -KeyLength 4096 -HashAlgorithm SHA256 -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" -ValidityPeriod Years -ValidityPeriodUnits 20 -Force
```

### Post Configuration of Offline Root CA

```powershell
$crllist = Get-CACrlDistributionPoint; foreach ($crl in $crllist) {Remove-CACrlDistributionPoint $crl.uri -Force};
Add-CACRLDistributionPoint -Uri C:\Windows\System32\CertSrv\CertEnroll\BEDROCK-ROOT%8%9.crl -PublishToServer -PublishDeltaToServer -Force
Add-CACRLDistributionPoint -Uri http://pki.bedrock.domain/pki/BEDROCK-ROOT%8%9.crl -AddToCertificateCDP -AddToFreshestCrl -Force
Get-CAAuthorityInformationAccess | where {$_.Uri -like '*ldap*' -or $_.Uri -like '*http*' -or $_.Uri -like '*file*'} | Remove-CAAuthorityInformationAccess -Force
Add-CAAuthorityInformationAccess -AddToCertificateAia http://pki.bedrock.domain/pki/BEDROCK-ROOT%3%4.crt -Force
certutil.exe –setreg CA\CRLPeriodUnits 20
certutil.exe –setreg CA\CRLPeriod “Years”
certutil.exe –setreg CA\CRLOverlapPeriodUnits 3
certutil.exe –setreg CA\CRLOverlapPeriod “Weeks”
certutil.exe –setreg CA\ValidityPeriodUnits 10
certutil.exe –setreg CA\ValidityPeriod “Years”
certutil.exe -setreg CA\AuditFilter 127
Restart-Service certsvc
```

### Publish new CRL

```powershell
certutil -crl
```

## Subordinate CA

### Publish Offline Root CA Certificate

```powershell
certutil.exe -dsPublish -f "C:\BEDROCK-ROOT.crl" RootCA
```

```powershell
certutil.exe –addstore –f root "C:\BEDROCK-ROOTBedrock Root Certificate Authority.crt"
certutil.exe –addstore –f root "C:\BEDROCK-ROOT.crl"
```

### Install ADCS Role for Subordinate CA

```powershell
Add-WindowsFeature Adcs-Cert-Authority -IncludeManagementTools
```

### Configure Subordinate CA

```powershell
Install-AdcsCertificationAuthority -CAType EnterpriseSubordinateCA -CACommonName "Bedrock Enterprise Certificate Authority" -KeyLength 4096 -HashAlgorithm SHA256 -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" -Force
```

### Retrieve signed Subordinate CA Certificate

```powershell
certreq -retrieve 2 "C:\issuingCACert.crt"
```

### Post Configuration of Suboardinate CA

```powershell
$crllist = Get-CACrlDistributionPoint; foreach ($crl in $crllist) {Remove-CACrlDistributionPoint $crl.uri -Force};
Add-CACRLDistributionPoint -Uri C:\Windows\System32\CertSrv\CertEnroll\BEDROCK-ECA%8%9.crl -PublishToServer -PublishDeltaToServer -Force
Add-CACRLDistributionPoint -Uri file://\\webserv1.bedrock.domain\pki\BEDROCK-ECA%8%9.crl -PublishToServer -PublishDeltaToServer -Force
Add-CACRLDistributionPoint -Uri http://pki.bedrock.domain/pki/BEDROCK-ECA%8%9.crl -AddToCertificateCDP -AddToFreshestCrl -Force
Get-CAAuthorityInformationAccess | where {$_.Uri -like '*ldap*' -or $_.Uri -like '*http*' -or $_.Uri -like '*file*'} | Remove-CAAuthorityInformationAccess -Force
Add-CAAuthorityInformationAccess -AddToCertificateAia http://pki.bedrock.domain/pki/BEDROCK-ECA%3%4.crt -Force
certutil.exe -setreg CA\CRLPeriodUnits 2
certutil.exe -setreg CA\CRLPeriod "Weeks"
certutil.exe -setreg CA\CRLDeltaPeriodUnits 1
certutil.exe -setreg CA\CRLDeltaPeriod "Days"
certutil.exe -setreg CA\CRLOverlapPeriodUnits 12
certutil.exe -setreg CA\CRLOverlapPeriod "Hours"
certutil.exe -setreg CA\ValidityPeriodUnits 5
certutil.exe -setreg CA\ValidityPeriod "Years"
certutil.exe -setreg CA\AuditFilter 127
restart-service certsvc
```

### Publish CRL

```powershell
certutil -crl
```

### Copy AIA .crt to the webserver

```powershell
copy "C:\Windows\System32\CertSrv\CertEnroll\issuingCA.bedrock.domain_Bedrock Enterprise Certificate Authority.crt" "\\WebServ1.bedrock.domain\pki\BEDROCK-ECABedrock Enterprise Certificate Authority.crt"
```
