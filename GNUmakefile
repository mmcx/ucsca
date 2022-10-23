PFXPASS = 123456

all: crtinfo chained.pem mycert.pfx.info

crt: rootca.crt signca.crt mycert.crt
crtinfo: rootca.crt.info signca.crt.info mycert.crt.info
csrinfo: rootca.csr.info signca.csr.info mycert.csr.info

cheeck: orig.pem.txt my.pem.txt

%.key.info: %.key
	openssl rsa -in $< -text -out $@

%.crt: %.p7b
	openssl pkcs7 -in $< -inform der -print_certs -out $@

%.crt.txt: %.crt
	openssl crl2pkcs7 -nocrl -certfile $< | openssl pkcs7 -print -out $@

%.crt.info: %.crt
	openssl x509 -text -nokey -in $< -out $@

%.csr.info: %.csr
	openssl req -text -noout -in $< -out $@


%.key:
	openssl genrsa -out $@ 2048

rootca.key:
	openssl genrsa -out $@ 4096


rootca.crt: rootca.key rootca.cnf GNUmakefile
	-rm rootca-index.txt
	touch rootca-index.txt
	mkdir -p rootca-certs
	openssl req -config rootca.cnf -new -nodes -key rootca.key -out rootca.csr
	openssl ca -config rootca.cnf -selfsign -batch -out rootca.pem -infiles rootca.csr
	openssl x509 -in rootca.pem -out rootca.crt -outform DER
	openssl ca -config rootca.cnf -gencrl -out rootca.crl.pem
	openssl crl -in rootca.crl.pem -out rootca.crl -outform DER

signca.crt: signca.key signca.cnf GNUmakefile
	-rm signca-index.txt
	touch signca-index.txt
	mkdir -p signca-certs
	openssl req -config signca.cnf -new -nodes -key signca.key -out signca.csr
	openssl ca -config signca.cnf -batch -out signca.pem -infiles signca.csr
	openssl x509 -in signca.pem -out signca.crt -outform DER
	openssl ca -config signca.cnf -gencrl -out signca.crl.pem
	openssl crl -in signca.crl.pem -out signca.crl -outform DER

mycert.crt: mycert.key mycert.cnf GNUmakefile
	-rm mycert-index.txt
	touch mycert-index.txt
	mkdir -p mycert-certs
	openssl req -config mycert.cnf -new -nodes -key mycert.key -out mycert.csr
	openssl ca -config mycert.cnf -batch -out mycert.pem -infiles mycert.csr
	openssl x509 -in mycert.pem -out mycert.crt -outform DER
	openssl ca -config mycert.cnf -gencrl -out mycert.crl.pem
	openssl crl -in mycert.crl.pem -out mycert.crl -outform DER

%.pem: %.crt
	openssl x509 -in $< -out $@

mycert.pfx: mycert.pem signca.pem rootca.pem GNUmakefile
	#openssl pkcs12 -export -out $@ -passout pass:$(PFXPASS) -inkey mycert.key -in mycert.crt -certfile chained.pem
	cat mycert.pem signca.pem rootca.pem > chained.pem
	#openssl pkcs12 -export -out mycert.pfx -in chained.pem -inkey mycert.key -passout 'pass:'
	openssl pkcs12 -export -certpbe PBE-SHA1-3DES -keypbe PBE-SHA1-3DES -nomac -out mycert.pfx -inkey mycert.key -in chained.pem -passout 'pass:$(PFXPASS)'

%.pfx.info: %.pfx GNUmakefile
	#openssl pkcs12 -in $< -passin 'pass:$(PFXPASS)' -passout 'pass:$(PFXPASS)' -out $@  -info
	openssl pkcs12 -in $< -passin 'pass:$(PFXPASS)' -passout 'pass:' -out $@  -info -noenc

#openssl req -x509 -new -nodes -key rootca.key -sha256 -days 9125 -out rootca.crt -config ./rootca.cnf
#openssl req -x509 -new -nodes -key signca.key -out signca.csr -CA rootca.crt -CAkey rootca.key  -config signca.cnf -extensions ca_extensions
#openssl req -text -noout -in signca.csr -out signca.csr.info
#rm index.txt || touch index.txt
#touch index.txt
#echo "01" > serial.txt
#openssl ca -batch -config signca.cnf -policy signing_policy -extensions signing_req -out signca.crt -infiles signca.csr
