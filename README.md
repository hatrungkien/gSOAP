#Set up gSOAP
```
//get libs
sudo apt-get install flex bison
sudo apt-get install libssl-dev
sudo apt-get install -y libz-dev

//move to home directory
cd

//download gSOAP Toolkit
wget "https://sourceforge.net/projects/gsoap2/files/gsoap_2.8.116.zip/download"

//unzip
unzip download

//now we have file gsoap-2.8 in home directory, lets setup
cd gsoap-2.8
./configure
cd gsoap
make
sudo make install

//install the executables in a local folder, back to gsoap-2.8 directory
cd ..
./configure
cd gsoap
make
sudo make install exec_prefix=$HOME

//The ./configure command takes the following configuration options:
cd ..
./configure --enable-samples
./configure --enable-ipv6
./configure --enable-ipv6-v6only
./configure --disable-ssl
./configure --enable-gnutls
./configure --enable-debug
./configure --disable-namespaces
./configure --disable-c-locale
./configure --enable-xlocale
./configure --with-openssl=DIR
./configure --with-zlib=DIR
```


#GENIVIA example
```asm
wsdl2h -o onvif.h -c -s -t ./typemap.dat http://www.onvif.org/onvif/ver10/network/wsdl/remotediscovery.wsdl
soapcpp2 -2 -I ~/gsoap-2.8/gsoap/import onvif.h
```
- -O4 aggressively optimizes the output by "schema slicing" to remove unused schema components, see our article Schema Slicing Methods to Reduce Development Costs of WSDL-Based Web Services for details;
- -P removes the base class xsd__anyType from the generated C++ classes, which are normally added by wsdl2h if the xsd:anyType XSD type is used somewhere in a WSDL. However, for the ONVIF protocols we do not need to inherit the xsd__anyType class and we can reduce the generated code size accordingly;
- -x removes the unnecessary generated code for the extensibility elements xsd:any and attributes xsd:anyAttribue, since we do not need to support these in general, except for some specific cases see further below. 
- -d to generate embedded DOM code which allows you to add any XML content via the generated xsd__anyType __any members that are DOM nodes;
- -o onvif.h saves the binding interface code to the header file onvif.h;
- -c to generate C source code instead of C++ source code, see further below.
- -2 forces SOAP 1.2, which is required by ONVIF;
- -I sets the import path to the ~/gsoap-2.8/gsoap/import directory in the gSOAP source code tree, note that this is an example and you should specify the actual location of the gSOAP directory on your system.

There is 2 types of duration: duration and chrono_duration add your choice in typemap.dat
```asm
//add duration
xsd__duration = #import "custom/duration.h" | xsd__duration
```
```asm
//add chrono_duration
xsd__duration = #import "custom/chrono_duration.h" | xsd__duration
```

compile main.cpp:
```asm
c++ -o ipcamera -Wall -DWITH_OPENSSL -DWITH_DOM -DWITH_ZLIB \
  -I. -I ~/gsoap-2.8/gsoap/plugin -I ~/gsoap-2.8/gsoap/custom -I ~/gsoap-2.8/gsoap \
  main.cpp \
  soapC.cpp \
  wsddClient.cpp \
  wsddServer.cpp \
  soapAdvancedSecurityServiceBindingProxy.cpp \
  soapDeviceBindingProxy.cpp \
  soapDeviceIOBindingProxy.cpp \
  soapImagingBindingProxy.cpp \
  soapMediaBindingProxy.cpp \
  soapPTZBindingProxy.cpp \
  soapPullPointSubscriptionBindingProxy.cpp \
  soapRemoteDiscoveryBindingProxy.cpp \
  ~/gsoap-2.8/gsoap/stdsoap2.cpp \
  ~/gsoap-2.8/gsoap/dom.cpp \
  ~/gsoap-2.8/gsoap/plugin/smdevp.c \
  ~/gsoap-2.8/gsoap/plugin/mecevp.c \
  ~/gsoap-2.8/gsoap/plugin/wsaapi.c \
  ~/gsoap-2.8/gsoap/plugin/wsseapi.c \
  ~/gsoap-2.8/gsoap/plugin/wsddapi.c \
  -lcrypto -lssl -lz
```



