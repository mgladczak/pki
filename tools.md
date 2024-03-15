# PKI Tools

Tool: [PKIView](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/quick-check-on-adcs-health-using-enterprise-pki-tool-pkiview/ba-p/1128638)

Weryfikacja: `certutil -verify C:\path\to\certificate.crt`


cd \certs
mkdir <fqdn>
cd .\<fqdn>

openssl genrsa -out <fqdn>.key 4096

Tworzymy plik zawierający szczegóły certyfikatu, zapisujemy jako plik z rozszerzeniem `.cnf`, sugerowana nazwa: `<fqdn>.cnf` w celu łatwiejszej identyfikacji

```powershell
  [ req ]
  default_bits = 4096
  distinguished_name = req_distinguished_name
  req_extensions  = req_ext
  prompt = no

  [ req_distinguished_name ]
  countryName                            = PL
  stateOrProvinceName                    = WP
  localityName                           = Poznan
  organizationName                       = RzepeckiMroczkowski
  commonName                             = host.domena
  [ req_ext ]
  subjectAltName = @alt_names
  [alt_names]
  DNS.1 = host1
  DNS.2 = host1.domena
  DNS.2 = host2.domena
  IP.1 = 10.10.10.13
  IP.2 = 10.10.10.14
  IP.3 = 10.10.10.17
```

openssl req -new -key <fqdn>.key -out <fqdn>.csr -config <fqdn>.cnf

certreq -submit -attrib "CertificateTemplate: RMWebServer" <fqdn>.csr

Na serwer kliencki wgrywamy następujące pliki:

- `<fqdn>.crt` - certyfikat wystawiony dla fqdn określonych w `[alt_names]` oraz `commonName` w pliku `<fqdn>.cnf`
- `<fqdn>.key` - klucz prywatny wytawiony dla fqdn określonych w `[alt_names]` oraz `commonName` w pliku `<fqdn>.cnf`

openssl x509 -inform der -in <certyfikat_der> -outform pem -out <certyfikat_pem>
