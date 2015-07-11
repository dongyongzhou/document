---
layout: master
title: Network Security
---

## 常规加密技术	
## 公开密钥密码学	
## 数字签名和认证技术
### How to generate SHA256 Signed Certificates 
NOTE:  uses different openSSL extension files (v3.ext, v3_attest.ext) when generating the attestation CA and attestation certificate. Misuse of openSSL extension files may lead to failure due to an invalid certificate chain.
Please use opensslroot.cfg v3.ext  and v3_attest.ext. 
 
1.Generate root CA key and generate root Certificate Signing Request and decode Root CA certificates CSR

	openssl genrsa -out dy_rootca.key -3 2048
	openssl req -new -key dy_rootca.key -x509 -out dy_rootca.crt -SHA256 -subj /C=CN/ST=SZ/L="Shenzhen"/OU="General Use Test Key (for testing only)"/OU="dongyong tech"/O=DY/CN="DY Root CA 1" -days 7300 -set_serial 1 -config opensslroot.cfg
	openssl req -in dy_rootca.crt -noout -text
	
	openssl genrsa -out dy_attestca.key -3 2048 
	openssl req -new -key dy_attestca.key -out dy_attestca.csr -SHA256 -subj /C=CN/ST=SZ/L="Shenzhen"/OU="dongyong Tech"/O=DY/CN="DY Attestation CA" -days 7300 -config opensslroot.cfg
	
	openssl x509 -req -in dy_attestca.csr -CA dy_rootca.crt -CAkey dy_rootca.key -out dy_attestca.crt -SHA256 -set_serial 5 -days 7300 -extfile v3.ext
	
	openssl genrsa -out dy_attest.key -3 2048
	
	openssl req -new -key dy_attest.key -out dy_attest.csr -SHA256 -subj  /C=CN/ST=SZ/L="Shenzhen"/emailAddress=dongyong800@163.com/OU="07 0001 DIGEST"/OU="06 0001 MODEL_ID"/OU="05 00002000  SW_SIZE"/OU="04 0001 OEM_ID"/OU="03 000000000000000F DEBUG"/OU="02 007180E100010001 HW_ID"/OU="01 000000000000000  SW_ID"/O=DY/CN="DY Attestation Cert" -days 7300 -config  opensslroot.cfg
	
	openssl x509 -req -in dy_attest.csr -CA dy_attestca.crt -CAkey  dy_attestca.key -out fsb_attest.crt -SHA256 -set_serial 7 -days 7300  -extfile v3_attest.ext  
	openssl x509 -in dy_rootca.crt -inform PEM -out dy_rootca.cer -outform  DER 
	openssl x509 -in dy_attestca.crt -inform PEM -out dy_attestca.cer -outform DER
	openssl dgst -sha256 dy_rootca.cer

View and verify CA certificates

	openssl x509 -noout -text -in dy_rootca.cer
	
	certificate chain has the following files from above steps: 
	opensslroot.cfg 
	dy_attestca.cer
	dy_attestca.key
	dy_rootca.cer
	dy_rootca.key
	v3_attest.ext v3.ext 


### 

