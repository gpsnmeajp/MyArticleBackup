Windowsの通知が、Windows 8.1からバルーンの代わりにトーストになり、通知センターに記録されるようになりました。
今まで通知の中身を取り出すには、Windowsのハンドルを捕まえて読み出そうとしたりなんだりしないといけないもので、
特にトーストになって以降はそういったことも難しくなりましたが、実はAnniversary Updateの時点で
Androidの通知を取得するのと同じような感覚でWindowsの通知が取得できる『通知リスナー』が実装されていました。

**2021/06/27追記: このAPIで受け取ってWebsocketで配信するツール作りました。.NET 5では少し楽に取得できるようになっています。**

https://github.com/gpsnmeajp/NotificationListenerThrower

通知リスナー: すべての通知にアクセスする - Windowsデベロッパーセンター
https://docs.microsoft.com/ja-JP/windows/uwp/design/shell/tiles-and-notifications/notification-listener

これはUWPアプリケーション向けの機能ですが、uwpdesktopを利用することでデスクトップアプリケーションからも取得できました。

uwpdesktopの使い方については以下の記事を参照ください。

Windows10のC#コンソールアプリケーションでBLEのアドバタイズメントをスキャンしたい
https://qiita.com/gpsnmeajp/items/607959d9eb76f908ef25

以下のように取得できます。
(テスト用コードに更新時に取得する処理をまだ実装していないので、現在のを延々と取得する処理になっています。)

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/e1ef15d5-f62e-0403-52aa-8b923c0abd27.png)

コードは以下です。
MSのサンプルソースほぼそのままです。

```C#
using System;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Windows.UI.Notifications.Management;
using Windows.Foundation.Metadata;
using System.Collections.Generic;
using Windows.UI.Notifications;


namespace ConsoleApplication2
{
    class Program
    {

        static int Main(string[] args)
        {
            //コンソールアプリケーションからAsyncを呼び出す大本はTaskを使用する;
            return MainAsync().Result;
        }

        //AsyncなMain
        private static async Task<int> MainAsync()
        {
            if (!ApiInformation.IsTypePresent("Windows.UI.Notifications.Management.UserNotificationListener"))
            {
                Console.WriteLine("IsTypePresent: NG");
                return -1;
            }
            Console.WriteLine("IsTypePresent: OK");

            UserNotificationListener listener = UserNotificationListener.Current;
            Console.Write("listener: ");
            Console.WriteLine(listener);

            UserNotificationListenerAccessStatus accessStatus = await listener.RequestAccessAsync();

            Console.Write("accessStatus: ");
            Console.WriteLine(accessStatus);

            if (accessStatus != UserNotificationListenerAccessStatus.Allowed)
            {
                Console.WriteLine("アクセス拒否");
                return -1;
            }
            Console.WriteLine("アクセス許可");


            while(true)
            {
                IReadOnlyList<UserNotification> notifs = await listener.GetNotificationsAsync(NotificationKinds.Toast);

                foreach (var n in notifs) {
                    NotificationBinding toastBinding = n.Notification.Visual.GetBinding(KnownNotificationBindings.ToastGeneric);

                    if (toastBinding != null)
                    {
                        IReadOnlyList<AdaptiveNotificationText> textElements = toastBinding.GetTextElements();

                        string titleText = textElements.FirstOrDefault()?.Text;

                        string bodyText = string.Join("\n", textElements.Skip(1).Select(t => t.Text));

                        Console.Write("Title:");
                        Console.WriteLine(titleText);
                        Console.Write("Body:");
                        Console.WriteLine(bodyText);   
                    }

                    Thread.Sleep(1000);
                }

            }

            return 0;
        }
    }
}

```

新しい通知を選別する。
イベント機能は事実上利用できないのと、公式に差分を取るしか無い旨が書いてある

```C#
using System;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Windows.UI.Notifications.Management;
using Windows.Foundation.Metadata;
using System.Collections.Generic;
using Windows.UI.Notifications;
using Windows.Foundation;
using Windows.ApplicationModel.Background;

namespace ConsoleApplication2
{
    

    class Program
    {
        static UserNotificationListener listener;
        static int Main(string[] args)
        {
            //コンソールアプリケーションからAsyncを呼び出す大本はTaskを使用する;
            return MainAsync().Result;
        }

       

        //AsyncなMain
        private static async Task<int> MainAsync()
        {
            if (!ApiInformation.IsTypePresent("Windows.UI.Notifications.Management.UserNotificationListener"))
            {
                Console.WriteLine("IsTypePresent: NG");
                return -1;
            }
            Console.WriteLine("IsTypePresent: OK");

            // And request access to the user's notifications (must be called from UI thread)
            listener = UserNotificationListener.Current;
            UserNotificationListenerAccessStatus accessStatus = await listener.RequestAccessAsync();

            Console.Write("accessStatus: ");
            Console.WriteLine(accessStatus);

            if (accessStatus != UserNotificationListenerAccessStatus.Allowed)
            {
                Console.WriteLine("アクセス拒否");
                return -1;
            }
            Console.WriteLine("アクセス許可");


            while (true)
            {
                newnotice();
                Thread.Sleep(1000);
            }

            return 0;
        }


        private static void show(UserNotification u)
        {
            NotificationBinding toastBinding = u.Notification.Visual.GetBinding(KnownNotificationBindings.ToastGeneric);

            if (toastBinding != null)
            {
                // And then get the text elements from the toast binding
                IReadOnlyList<AdaptiveNotificationText> textElements = toastBinding.GetTextElements();

                // Treat the first text element as the title text
                string titleText = textElements.FirstOrDefault()?.Text;

                // We'll treat all subsequent text elements as body text,
                // joining them together via newlines.
                string bodyText = string.Join("\n", textElements.Skip(1).Select(t => t.Text));

                Console.Write("Title:");
                Console.WriteLine(titleText);
                Console.Write("Body:");
                Console.WriteLine(bodyText);
            }

        }

        static List<uint> notificationIds = new List<uint>(); //通知リスト
        private static async void newnotice()
        {
            IReadOnlyList<UserNotification> userNotifications = await listener.GetNotificationsAsync(NotificationKinds.Toast);
            List<uint> notificationIdsNow = new List<uint>();

            //新しい通知を検出する
            foreach (UserNotification n in userNotifications)
            {
                if (!notificationIds.Contains(n.Id))
                {
                    //新しい通知だ！
                    Console.WriteLine("------------- Add! : " + n.Id + "------------- ");
                    show(n);
                    notificationIds.Add(n.Id);
                }
                //現在の通知を記録していく
                notificationIdsNow.Add(n.Id);
            }

            //存在しなくなった通知を検出する
            List<uint> removeIdList = new List<uint>();
            foreach (uint id in notificationIds)
            {
                if (!notificationIdsNow.Contains(id))
                {
                    //消えた通知だ！
                    Console.WriteLine("------------- Remove! : " + id + "------------- ");
                    removeIdList.Add(id);
                }
            }

            //foreach中に削除できないので一旦別のに入れて削除している
            foreach (uint id in removeIdList)
            {
                notificationIds.Remove(id);
            }
        }
    }
}
```
