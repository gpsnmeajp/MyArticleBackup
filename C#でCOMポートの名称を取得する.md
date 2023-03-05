PnPデバイス一覧から取り出して、マッチングします。

```
通信ポート (COM1) is COM1
USB シリアル デバイス (COM4) is COM4
```

のように出力が出ます。Dictionaryなどに入れれば、ユーザーにわかりやすい表示になると思います。

Portsでフィルタすると、シリアルポートとパラレルポートまで絞れます。
Serviceでフィルタしようとすると、本体ポートとUSBポートで違い、ドライバによっても差が出そうなので止めました。

```C#
            //PnPデバイスを列挙
            ManagementClass managementClass = new ManagementClass("Win32_PnPEntity");
            foreach (var i in managementClass.GetInstances()) {
                //Portsを持つデバイスのみ抽出
                if ((string)i.GetPropertyValue("PNPClass") == "Ports")
                {
                    string name = (string)i.GetPropertyValue("name");
                    Match match = Regex.Match(name, "\\((COM\\d+)\\)");
                    if (match.Success && match.Groups.Count > 1)
                    {
                        string port = match.Groups[1].Value;
                        Console.WriteLine(name + " is " + port);
                    }

                    //foreach (var k in i.Properties)
                    //{
                    //  Console.WriteLine(k.Name + ":" + k.Value);
                    //}
                }
            }
```

# 参考文献
