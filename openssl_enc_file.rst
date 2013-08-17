openssl对文件进行非对称密钥加密和解密
##########################################################################################################################################
:date: 2011-02-24 09:32:19
:category: 系统运维
:slug: openssl_enc_file

#生成私钥 

 openssl genrsa > rsaprivatekey.pem 

#生成公钥 

 openssl rsa -pubout < rsaprivatekey.pem > rsapubckey.pem 

#生成测试文件 

 echo test > if.txt 

#使用公钥加密 

 openssl rsautl -encrypt -pubin -inkey rsapubckey.pem < if.txt > test-encrypted.txt 

#使用私钥解密 
 openssl rsautl -decrypt -inkey rsaprivatekey.pem < test-encrypted.txt > /dev/stdout 

注意：这种方式不能加密大一点儿的文件，否则会出现以下的错误: 

 RSA operation error 
 3074250376:error:0406D06E:rsa
 routines:RSA\_padding\_add\_PKCS1\_type\_2:data too large for key
 size:rsa\_pk1.c:151: 
