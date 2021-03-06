# Charles Proxy Mobile Guide
The mobile hackers' guide to Charles Proxy :thumbsup:

## Intro

[Charles](https://www.charlesproxy.com/) is an HTTP proxy / HTTP monitor / Reverse Proxy that enables a developer to view all of the HTTP and SSL / HTTPS traffic between their machine and the Internet. This includes requests, responses and the HTTP headers (which contain the cookies and caching information).

## SSL Proxying

### Computer (macOS)

* **Charles** -> **Proxy** -> **SSL Proxying Settings...** -> **SSL Proxying**
    * Check "**Enable SSL Proxying**"
    * Add location matcher "**Host: &ast;**", "**Port: &ast;**"
    
    <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/macOS/SSL-Proxying-Settings.png" width="600">
    
* **Charles** -> **Help** -> **SSL Proxying** -> **Install Charles Root Certificate**
    * It would install the certificate to **Keychain**, and open it up automatically
    * Double click the certificate in Keychain
    * Expand **Trust**, select **Always Trust** for **When using this certificate**
    
    <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/macOS/Root-Certificate-not-trusted.png" width="400">
    
    <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/macOS/Root-Certificate-trusted.png" width="400">
    
* Get the **IP address** that Charles is listening to
    * **Charles** -> **Help** -> **Local IP Address**
    * Or get the WLAN IP address via command line (right into your clipboard)
        ```shell
        ifconfig | tr "\n" "→" | tr "\r" "→" | grep -Eo "→en0.*?→en[[:digit:]]" | grep -Eo "inet[[:blank:]+]([0-9]{1,3}\.){3}[0-9]{1,3}" | cut -d' ' -f2 | tr -d "\n" | pbcopy && pbpaste
        ```
    * Or get the LAN IP address via command line (right into your clipboard)
        ```shell
        ifconfig | tr "\n" "→" | tr "\r" "→" | grep -Eo "→en[[:digit:]].*?active→" | grep -v "en0" | grep -Eo "inet[[:blank:]+]([0-9]{1,3}\.){3}[0-9]{1,3}" | cut -d' ' -f2 | tr -d "\n" | pbcopy && pbpaste
        ```

### Android

* Launch **Charles** and keep it running
* Get the **IP address**
* Make sure the Android device uses the same network as Charles
* On **Android** device
    * Go to **Settings** -> **Wi-Fi** -> long click the **network** in use -> **Modify network** -> **Advanced options** -> **Proxy** -> **Manual**
        * **Proxy hostname** = **IP address**
        * **Proxy port** = **8888**
        
        <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/Android/Wi-Fi.png" width="256">
    
    * Launch **Browser**, visit https://chls.pro/ssl, save the certificate
    
    <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/Android/certificate.png" width="256">
    
    * The system would ask you to set a lock screen **PIN** or **password**, just set one and save it
    * Now the certificate is installed
    * Open an application and monitor the traffic on Charles
* A dialog pops up on computer asking "A connection attempt to  Charles has been made from the host ...", just click **Allow** button

#### Android N (7.0, API level 24) and afterwards

* Open your Android project with Android Studio
* **Android Studio** -> **File** -> **New** -> **Android resource directory**
    * **Directory name** = **xml**
    * **Directory type** = **xml**
    * **Source set** = **debug**
* **Android Studio** -> **File** -> **New** -> **XML resource file**
    * **File name** = **network_security_config**
    * **Root element** = **network-security-config**
* Above step would generate a XML file with the given root element.  Now paste below content to replace the existing content in the generated XML file.
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config xmlns:android="http://schemas.android.com/apk/res/android">
        <debug-overrides>
            <trust-anchors>
                <!-- Trust user added CAs while debuggable only -->
                <certificates src="user" />
            </trust-anchors>
        </debug-overrides>
    </network-security-config>
    ```
* Then go to **debug** source set, create a blank **AndroidManifest.xml** file if you don't have one for the debug build variant, and add content like below (eventually the manifest merger will merge it with the main manifest).  When you already have one, simply add the `networkSecurityConfig` attribute under `application`.
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools">
    
        <application
            android:networkSecurityConfig="@xml/network_security_config">
        </application>
    
    </manifest>
    ```

Now the SSL proxying should work for your app's debug build variant, but not for release build variant.

### iOS

* Launch **Charles** and keep it running
* Get the **IP address**
* Make sure the iOS device uses the same network as Charles
* On **iOS** device
    * Go to **Settings** -> **Wi-Fi** -> click the **network** in use -> set **HTTP PROXY** to **Manual**
        * **Server** = **IP address**
        * **Port** = **8888**
        
        <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/iOS/Wi-Fi.png" width="256">
    
    * Launch **Safari**, visit https://chls.pro/ssl (if it doesn't work, try http://www.charlesproxy.com/getssl), install the certificate
    
    <img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/iOS/certificate.png" width="256">
    
    * Open an application and monitor the traffic on Charles
* A dialog pops up on computer asking "A connection attempt to  Charles has been made from the host ...", just click **Allow** button

#### iOS 10.3 and afterwards

* After the certificate is installed
* Go to **Settings** -> **General** -> **About** -> **Certificate Trust Settings** -> **Enable Full Trust For Root Certificates** -> enable Charles' certificate

<img src="https://github.com/thyrlian/Charles-Proxy-Mobile-Guide/blob/master/Screenshots/iOS/Certificate-Trust-Settings.png" width="256">

### Terminal

In case you need to debug via curl in a terminal:

* To set Charles as the proxy

    ```shell
    export http_proxy=http://127.0.0.1:8888 && export https_proxy=$http_proxy
    ```

* To remove the proxy

    ```shell
    unset http_proxy https_proxy
    ```

### SSL CA Certificate

* Charles Proxy running on the same computer provides the same SSL certificate.
