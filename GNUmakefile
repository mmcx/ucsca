PFXPASS = 123456

CRLOPTS+=-crl_lastupdate  20221028000000Z
CRLOPTS+=-crl_nextupdate  20221128000000Z

ROOTCA_CAOPTS+=-startdate 20221028000000Z
ROOTCA_CAOPTS+=-enddate   20471028000000Z

SIGNCA_CAOPTS+=-startdate 20221028000000Z
SIGNCA_CAOPTS+=-enddate   20321028000000Z

MYCERT_CAOPTS+=-startdate 20221028000000Z
MYCERT_CAOPTS+=-enddate   20251028000000Z



all: mycert.pfx.info host

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
	mkdir -p $(diname $@)
	openssl genrsa -out $@ 2048

rootca.key:
	mkdir -p $(diname $@)
	openssl genrsa -out $@ 4096

rootca.crt: rootca.key rootca.cnf GNUmakefile
	mkdir -p temp
	mkdir -p certs
	-rm temp/rootca-index.txt
	touch temp/rootca-index.txt
	echo '77:f5:71:77:01:5a:d7:64:47:32:71:85:08:fa:cf:54:b0:ec:38:1b' > temp/rootca-serial.txt
	echo '77:f5:71:77:01:5a:d7:64:47:32:71:85:08:fa:cf:54:b0:ec:38:1b' | tr -d ':' > temp/rootca-serial.txt
	echo '01' | tr -d ':' > temp/rootca-serial.txt
	openssl req -config rootca.cnf -new -nodes -key rootca.key -out temp/rootca.csr
	openssl req -text -noout -in temp/rootca.csr -out temp/rootca.csr.info
	openssl ca -config rootca.cnf -selfsign -batch $(ROOTCA_CAOPTS) -out temp/rootca.pem -infiles temp/rootca.csr
	openssl x509 -in temp/rootca.pem -out rootca.crt -outform DER
	openssl ca -config rootca.cnf -gencrl $(CRLOPTS) -out temp/rootca.crl.pem
	openssl crl -in temp/rootca.crl.pem -out rootca.crl -outform DER
	

signca.crt: signca.key signca.cnf rootca.crt GNUmakefile
	mkdir -p temp
	-rm temp/signca-index.txt
	touch temp/signca-index.txt
	echo '0c:f5:52:bd:32:c0:2d:0f:d3:2d:42:c2:39:91:2c:6e:a3:6f:a7:c2' > temp/signca-serial.txt
	openssl req -config signca.cnf -new -nodes -key signca.key -out temp/signca.csr
	openssl ca -config signca.cnf -batch $(SIGNCA_CAOPTS) -out temp/signca.pem -infiles temp/signca.csr
	openssl x509 -in temp/signca.pem -out signca.crt -outform DER
	openssl ca -config signca.cnf -gencrl $(CRLOPTS) -out temp/signca.crl.pem
	openssl crl -in temp/signca.crl.pem -out signca.crl -outform DER

host: rootca.crt signca.crt
	cp rootca.crt CompanyUCSRootCA.crt
	cp rootca.crl CompanyUCSRootCA.crl
	cp signca.crt CompanyUCSCodeSigningCA.crt
	cp signca.crl CompanyUCSCodeSigningCA.crl

mycert.crt: mycert.key mycert.cnf signca.crt GNUmakefile
	mkdir -p temp/
	-rm temp/mycert-index.txt
	touch temp/mycert-index.txt
	echo '77:f5:71:77:01:5a:d7:64:47:32:71:85:08:fa:cf:54:b0:ec:38:1b' > temp/mycert-serial.txt
	openssl req -config mycert.cnf -new -nodes -key mycert.key -out temp/mycert.csr
	openssl ca -config mycert.cnf -batch $(MYCERT_CAOPTS) -out temp/mycert.pem -infiles temp/mycert.csr
	openssl ca -config mycert.cnf -gencrl $(CRLOPTS) -out temp/mycert.crl.pem
	openssl crl -in temp/mycert.crl.pem -out mycert.crl -outform DER

mycert.pfx: mycert.crt signca.crt rootca.crt GNUmakefile
	mkdir -p temp
	openssl x509 -in rootca.crt -out temp/rootca.pem
	openssl x509 -in signca.crt -out temp/signca.pem
	cat temp/rootca.pem temp/signca.pem temp/mycert.pem > temp/chained.pem
	openssl pkcs12 -export -certpbe PBE-SHA1-3DES -keypbe PBE-SHA1-3DES -nomac -out mycert.pfx -inkey mycert.key -in temp/chained.pem -passout 'pass:$(PFXPASS)'

%.pfx.info: %.pfx GNUmakefile
	openssl pkcs12 -in $< -passin 'pass:$(PFXPASS)' -passout 'pass:$(PFXPASS)' -out $@  -info -noenc

#openssl req -x509 -new -nodes -key rootca.key -sha256 -days 9125 -out rootca.crt -config ./rootca.cnf
#openssl req -x509 -new -nodes -key signca.key -out signca.csr -CA rootca.crt -CAkey rootca.key  -config signca.cnf -extensions ca_extensions
#openssl req -text -noout -in signca.csr -out signca.csr.info
#rm index.txt || touch index.txt
#touch index.txt
#echo "01" > serial.txt
#openssl ca -batch -config signca.cnf -policy signing_policy -extensions signing_req -out signca.crt -infiles signca.csr
