# はじめに
Misskey v13.6.1 でのコードです。
misskey.io (2023/02/26)で動作を確認しています。

APIが不安定とのことで、将来的には動作しないかもしれません。

# コード
C#のHttpClient.PostAsyncでアクセスします。
画像投稿周りは公式のAPIマニュアルにもあまり乗っていないので、公式Webクライアントの動作を参考に実装しました。

一旦動けば良いコードとして作成しているので、json周りの処理はグダグダです。
ただ、これで動くということは、相応に作れば相応に動くとうことですので、気軽に参考にしてください。

ライセンスはCC0としてください。

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Net;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;

namespace MiAPI
{
    class Program
    {
        static HttpClient client = new HttpClient();
        const string baseUrl = "https://misskey.io/api";
        const string accessToken = "xXxXxXxXxXxXxXxXxXx YOUR API KEY xXxXxXxXxXxXxXxXxXx";

        static void Main(string[] args)
        {
            Task.Run(async () =>
            {
                //Console.WriteLine(await GetTimeline());
                string file_id = await UploadImage();
                await CreateNote("こんにちは、これはAPIからの画像添付付き投稿テストです", file_id);
            }).Wait();
        }

        static async Task CreateNote(string text, string fileid)
        {
            var json_in = "{\"i\":\"" + accessToken + "\", \"visibility\": \"home\" ,\"text\": \""+text+"\"}";
            if (!String.IsNullOrEmpty(fileid)) {
                json_in = "{\"i\":\"" + accessToken + "\", \"visibility\": \"home\" ,\"text\": \"" + text + "\",\"fileIds\": [\"" + fileid + "\"]}";
            }

            var content = new StringContent(json_in, Encoding.UTF8, @"application/json");

            Console.WriteLine("API Call...");

            var responce = await client.PostAsync(baseUrl + "/notes/create", content);

            Console.WriteLine(responce.StatusCode);

            var responce_text = await responce.Content.ReadAsStringAsync();
            var json_out = JsonConvert.DeserializeObject<dynamic>(responce_text);
            Console.WriteLine(json_out);
        }


        static async Task<string> UploadImage()
        {
            var content = new MultipartFormDataContent();

            //アクセストークン
            var iContent = new StringContent(accessToken, new UTF8Encoding(false));
            iContent.Headers.ContentDisposition = new System.Net.Http.Headers.ContentDispositionHeaderValue("form-data");
            iContent.Headers.ContentDisposition.Name = "i";
            content.Add(iContent);

            //force
            //var forceContent = new StringContent("true", new UTF8Encoding(false));
            //forceContent.Headers.ContentDisposition = new System.Net.Http.Headers.ContentDispositionHeaderValue("form-data");
            //forceContent.Headers.ContentDisposition.Name = "force";
            //content.Add(forceContent);

            //Data
            var dataContent = new ByteArrayContent(File.ReadAllBytes(@"C:\sample_jpeg.jpg"));
            dataContent.Headers.ContentDisposition = new System.Net.Http.Headers.ContentDispositionHeaderValue("form-data");
            dataContent.Headers.ContentDisposition.Name = "file";
            dataContent.Headers.ContentDisposition.FileName = "helloworld3.jpg";
            dataContent.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue("image/jpeg");
            content.Add(dataContent);

            //name
            var nameContent = new StringContent("helloworld3.jpg", new UTF8Encoding(false));
            nameContent.Headers.ContentDisposition = new System.Net.Http.Headers.ContentDispositionHeaderValue("form-data");
            nameContent.Headers.ContentDisposition.Name = "name";
            content.Add(nameContent);

            Console.WriteLine("API Call...");
            var responce = await client.PostAsync(baseUrl + "/drive/files/create", content);

            Console.WriteLine(responce.StatusCode);
            var responce_text = await responce.Content.ReadAsStringAsync();
            var json_out = JsonConvert.DeserializeObject<dynamic>(responce_text);
            Console.WriteLine(json_out);

            return (json_out as Newtonsoft.Json.Linq.JContainer).Value<string>("id");
        }

        static async Task<string> GetTimeline() {
            var json_in = "{\"i\":\"" + accessToken + "\"}";
            var content = new StringContent(json_in, Encoding.UTF8, @"application/json");

            Console.WriteLine("API Call...");

            var responce = await client.PostAsync(baseUrl + "/notes/timeline", content);

            Console.WriteLine(responce.StatusCode);


            var responce_text = await responce.Content.ReadAsStringAsync();
            var json_out = JsonConvert.DeserializeObject<dynamic>(responce_text);

            string ret = "";

            foreach (var d in (json_out as Newtonsoft.Json.Linq.JArray))
            {
                string text = d.Value<string>("text");
                string username = d.Value<Newtonsoft.Json.Linq.JContainer>("user").Value<string>("username");
                if (text == null)
                {
                    var renote = d.Value<Newtonsoft.Json.Linq.JContainer>("renote");
                    var username_rt = renote.Value<Newtonsoft.Json.Linq.JContainer>("user").Value<string>("username");
                    text = "RT(@" + username_rt + "):" + renote.Value<string>("text");
                }

                var createdAtUTC = DateTime.Parse(d.Value<string>("createdAt"));
                var createdAt = TimeZoneInfo.ConvertTimeFromUtc(createdAtUTC, TimeZoneInfo.FindSystemTimeZoneById("Tokyo Standard Time"));

                //Console.WriteLine(d);
                ret += (username)+ "\n";
                ret += (createdAt)+ "\n";
                ret += (text)+ "\n";
                ret += "\n";
            }
            return ret;
        }
    }
}
```
