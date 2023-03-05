kshoji: [Android Advent Calendar] 自作のUSBデバイスを、Androidで動かす http://blog.kshoji.jp/2012/12/android-usb-host.html?spref=tw

ここのを試す時に、ベンダークラスだとどうすりゃいいんだろう、というか、
ベンダーIDとかで識別できたらなぁ、と思った。

https://developer.android.com/guide/topics/connectivity/usb/host.html

うん、普通にできる

```xml:device_filter.xml
<?xml version="1.0" encoding="utf-8"?>

<resources>
    <usb-device vendor-id="1240" product-id="83" />
</resources>
```
