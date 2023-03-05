最低限のライフライクルとそのログ、ボタンハンドラとテキスト書き換えのテンプレ

```java:MainActivity.java

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private String debugMsgTitle = "HELLO_WORLD_APP";
    private TextView mText;
    private Button mButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(debugMsgTitle,"Create");
        setContentView(R.layout.activity_main);

        mText = (TextView) findViewById(R.id.text);

        mButton = (Button) findViewById(R.id.button);
        mButton.setOnClickListener(this);
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.d(debugMsgTitle,"Resume");
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.d(debugMsgTitle,"Pause");
    }

    public void onClick(View v) {
        Button btn = (Button)v;
        Log.d(debugMsgTitle,"onClick");

        switch( btn.getId() ){
            //ボタンが押されたとき
            case R.id.button:
                mText.setText("Wow!");
                break;
            default:
                break;
        }
    }

}

```
