package com.example.samplevideoview;

import java.io.File;

import android.app.Activity;
import android.os.Bundle;
import android.widget.RelativeLayout;

import com.example.custom.view.CustomVideoView;

public class MainActivity extends Activity {

	private CustomVideoView videoView = null;
	private RelativeLayout relativeLayout = null;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		relativeLayout = (RelativeLayout) findViewById(R.id.main_relative_layout);

		RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(
				300, 300);
		params.leftMargin = 50;
		params.topMargin = 50;

		videoView = new CustomVideoView(this, new File("/sdcard/1.mp4"));
		// videoView = new CustomVideoView(this, "/sdcard/1.mp4");
		// videoView = new CustomVideoView(this,
		// Uri.parse("android.resource://com.example.samplevideoview/" +
		// R.raw.video));
		relativeLayout.addView(videoView, params);

	}

}
