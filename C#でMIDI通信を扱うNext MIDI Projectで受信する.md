C#でMIDI通信を扱う

Next MIDI Project (MITライセンス)
http://starway.s234.xrea.com/wordpress/

にて、MIDIを受信するサンプル

このコードはCC0ライセンスとして公開

```C#
using System;
using NextMidi.MidiPort.Input.Core;
using NextMidi.MidiPort.Output.Core;
using NextMidi.MidiPort.Input;
using Avenue;
using NextMidi.DataElement;
using System.Threading;

//http://starway.s234.xrea.com/wordpress/
namespace MIDIControllerTestConsole
{
    class Program
    {
        //受信イベント
        static void MidiReceived(object sender, DataEventArgs<MidiEvent> e)
        {
            byte[] data = e.Value.ToNativeEvent(); //MIDI生値に変換
            foreach (var d in data) {
                Console.Write("{0:X2},",d);
            }
            Console.Write("\n");
        }

        static void Main(string[] args)
        {
            //ポートを開く
            var port = new MidiInPort("loopMIDI Port");
            try
            {
                port.Open();
            }
            catch
            {
                Console.WriteLine("ポートが見つかりません");
                return;
            }

            //受信イベント登録
            port.Received += MidiReceived;

            //なにかキーを押すと終了
            while (!Console.KeyAvailable) {
                Console.WriteLine("Keep-Alive");
                Thread.Sleep(1000);
            }
        }
    }
}

```
