---
# required metadata

title: Create a Wi-Fi profile with a pre-shared keytitleSuffix: "Intune Azure preview"
description: "Intune Azure preview: Use an Intune custom profile to create a Wi-Fi profile with a pre-shared key."
keywords:
author: lleonard-msft
ms.author: alleonar
manager: angrobe
ms.date: 05/15/2017
ms.topic: article
ms.prod:
ms.service: microsoft-intune
ms.technology:
ms.assetid: c6fd72a6-7dc8-48fc-9df1-db5627a51597

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: karanda
ms.suite: ems
#ms.tgt_pltfrm:
ms.custom: intune-azure


---
# Use a Microsoft Intune custom device profile to create a Wi-Fi profile with a pre-shared key
[!INCLUDE[azure_preview](./includes/azure_preview.md)]

Here's how to use Intune’s **Custom device profiles** to create a Wi-Fi profile with a pre-shared key. This topic also has an example of how to create an EAP-based Wi-Fi profile.

> [!NOTE]
-	You might find it easier to copy the code from a computer that connects to that network, as described below.
- For Android, you also have the option of using this [Android PSK Generator](http://johnathonb.com/2015/05/intune-android-pre-shared-key-generator/) provided by Johnathon Biersack.
-	You can add multiple networks and keys by adding more OMA-URI settings.
-  For iOS, use Apple Configurator on a Mac station to set up the profile. Alternatively, use this [iOS PSK Mobile Config Generator](http://johnathonb.com/2015/05/intune-ios-psk-mobile-config-generator/) provided by Johnathon Biersack.


1.	To create a Wi-Fi profile with a pre-shared key for Android or Windows or an EAP-based Wi-Fi profile, when you create a device profile choose **Custom** for that device platform rather than a Wi-Fi profile.

2.	Provide a name and description.
3.	Add a new OMA-URI setting:

   a.	Enter a name for this Wi-Fi network setting.

   b.	Enter a description of the OMA-URI setting or leave blank.

   c.	**Data Type**: Set to **String**.

   d.	**OMA-URI**:

    - **For Android**: ./Vendor/MSFT/WiFi/Profile/<SSID>/Settings
    - **For Windows**: ./Vendor/MSFT/WiFi/Profile/MyNetwork/WlanXml

    > [!NOTE]
Be sure to include the dot character at the beginning.

    SSID is the SSID for which you’re creating the policy. For example,
    `./Vendor/MSFT/WiFi/Profile/Hotspot-1/Settings`

  e. **Value Field** is where you paste your XML code. Here’s an example. Each value should be adapted to your network settings. See the comments section of the code for some pointers.
4. Choose **OK**, save, and then assign the policy.

    > [!NOTE]
    > This policy can only be assigned to user groups.

The next time each device checks in, the policy will be applied, and a Wi-Fi profile will be created on the device. The device will be able to connect to the network automatically.

## Android or Windows Wi-Fi profile

Here’s an example of the XML code for an Android or Windows Wi-Fi profile:

> [!IMPORTANT]
>
> `<protected>false</protected>`must be set to **false**, as **true** could cause device to expect an encrypted password and then try to decrypt it, which may result in a failed connection.
>
>  `<hex>53534944</hex>` should be set to the hexadecimal value of `<name><SSID of wifi profile></name>`.
>  Windows 10 devices may return a false *0x87D1FDE8 Remediation failed* error, but will still be provisioned with the profile.

```
<!--
<Name of wifi profile> = Name of profile
<SSID of wifi profile> = Plain text of SSID. Does not need to be escaped, could be <name>Your Company's Network</name>
<nonBroadcast><true/false></nonBroadcast>
<Type of authentication> = Type of authentication used by the network, such as WPA2PSK.
<Type of encryption> = Type of encryption used by the network
<protected>false</protected> do not change this value, as true could cause device to expect an encrypted password and then try to decrypt it, which may result in a failed connection.
<password> = Password to connect to the network
x>53534944</hex> should be set to the hexadecimal value of <name><SSID of wifi profile></name>
-->
<WLANProfile
xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
  <name><Name of wifi profile></name>
  <SSIDConfig>
    <SSID>
      <hex>53534944</hex>
 <name><SSID of wifi profile></name>
    </SSID>
    <nonBroadcast>false</nonBroadcast>
  </SSIDConfig>
  <connectionType>ESS</connectionType>
  <connectionMode>auto</connectionMode>
  <autoSwitch>false</autoSwitch>
  <MSM>
    <security>
      <authEncryption>
        <authentication><Type of authentication></authentication>
        <encryption><Type of encryption></encryption>
        <useOneX>false</useOneX>
      </authEncryption>
      <sharedKey>
        <keyType>networkKey</keyType>
        <protected>false</protected>
        <keyMaterial>MyPassword</keyMaterial>
      </sharedKey>
      <keyIndex>0</keyIndex>
    </security>
  </MSM>
</WLANProfile>
```

## EAP-based Wi-Fi profile
Here’s  an example of the XML code for an EAP-based Wi-Fi profile:

```
    <WLANProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1">
      <name>testcert</name>
      <SSIDConfig>
        <SSID>
          <hex>7465737463657274</hex>
          <name>testcert</name>
        </SSID>
        <nonBroadcast>true</nonBroadcast>
      </SSIDConfig>
      <connectionType>ESS</connectionType>
      <connectionMode>auto</connectionMode>
      <autoSwitch>false</autoSwitch>
      <MSM>
        <security>
          <authEncryption>
            <authentication>WPA2</authentication>
            <encryption>AES</encryption>
            <useOneX>true</useOneX>
            <FIPSMode     xmlns="http://www.microsoft.com/networking/WLAN/profile/v2">false</FIPSMode>
          </authEncryption>
          <PMKCacheMode>disabled</PMKCacheMode>
          <OneX xmlns="http://www.microsoft.com/networking/OneX/v1">
            <cacheUserData>false</cacheUserData>
            <authMode>user</authMode>
            <EAPConfig>
              <EapHostConfig     xmlns="http://www.microsoft.com/provisioning/EapHostConfig">
                <EapMethod>
                  <Type xmlns="http://www.microsoft.com/provisioning/EapCommon">13</Type>
                  <VendorId xmlns="http://www.microsoft.com/provisioning/EapCommon">0</VendorId>
                  <VendorType xmlns="http://www.microsoft.com/provisioning/EapCommon">0</VendorType>
                  <AuthorId xmlns="http://www.microsoft.com/provisioning/EapCommon">0</AuthorId>
                </EapMethod>
                <Config xmlns="http://www.microsoft.com/provisioning/EapHostConfig">
                  <Eap xmlns="http://www.microsoft.com/provisioning/BaseEapConnectionPropertiesV1">
                    <Type>13</Type>
                    <EapType xmlns="http://www.microsoft.com/provisioning/EapTlsConnectionPropertiesV1">
                      <CredentialsSource>
                        <CertificateStore>
                          <SimpleCertSelection>true</SimpleCertSelection>
                        </CertificateStore>
                      </CredentialsSource>
                      <ServerValidation>
                        <DisableUserPromptForServerValidation>false</DisableUserPromptForServerValidation>
                        <ServerNames></ServerNames>
                      </ServerValidation>
                      <DifferentUsername>false</DifferentUsername>
                      <PerformServerValidation xmlns="http://www.microsoft.com/provisioning/EapTlsConnectionPropertiesV2">false</PerformServerValidation>
                      <AcceptServerName xmlns="http://www.microsoft.com/provisioning/EapTlsConnectionPropertiesV2">false</AcceptServerName>
                      <TLSExtensions xmlns="http://www.microsoft.com/provisioning/EapTlsConnectionPropertiesV2">
                        <FilteringInfo xmlns="http://www.microsoft.com/provisioning/EapTlsConnectionPropertiesV3">
                          <AllPurposeEnabled>true</AllPurposeEnabled>
                          <CAHashList Enabled="true">
                            <IssuerHash>75 f5 06 9c a4 12 0e 9b db bc a1 d9 9d d0 f0 75 fa 3b b8 78 </IssuerHash>
                          </CAHashList>
                          <EKUMapping>
                            <EKUMap>
                              <EKUName>Client Authentication</EKUName>
                              <EKUOID>1.3.6.1.5.5.7.3.2</EKUOID>
                            </EKUMap>
                          </EKUMapping>
                          <ClientAuthEKUList Enabled="true"/>
                          <AnyPurposeEKUList Enabled="false">
                            <EKUMapInList>
                              <EKUName>Client Authentication</EKUName>
                            </EKUMapInList>
                          </AnyPurposeEKUList>
                        </FilteringInfo>
                      </TLSExtensions>
                    </EapType>
                  </Eap>
                </Config>
              </EapHostConfig>
            </EAPConfig>
          </OneX>
        </security>
      </MSM>
    </WLANProfile>
```

## Create the XML file from an existing Wi-Fi connection
You can also create an XML file from an existing Wi-Fi connection:
1. On a computer that is connected to or has recently connected to the wireless network, open the following folder: C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces\{guid}.

    It’s best to use a computer that has not connected to many wireless networks, because you’ll have to search through each profile to find the right one.
3.     Search through the XML files to locate the one with the right name.
4.     After you have located the correct XML file, copy and paste the XML code into the Data field of the OMA-URI settings page.
