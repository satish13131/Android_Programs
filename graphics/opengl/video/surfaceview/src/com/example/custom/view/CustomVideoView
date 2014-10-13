package com.example.custom.view;

import java.io.File;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.nio.FloatBuffer;

import javax.microedition.khronos.egl.EGLConfig;
import javax.microedition.khronos.opengles.GL10;

import android.content.Context;
import android.graphics.PixelFormat;
import android.graphics.SurfaceTexture;
import android.media.MediaPlayer;
import android.net.Uri;
import android.opengl.GLES20;
import android.opengl.GLSurfaceView;
import android.opengl.Matrix;
import android.util.Log;
import android.view.Surface;

public class CustomVideoView extends GLSurfaceView {

	VideoRender mRenderer;
	private MediaPlayer mMediaPlayer = null;
	private File file = null;
	private String filePath = null;
	private Uri uri = null;

	public CustomVideoView(Context context, File file) {
		super(context);

		this.file = file;

		init();

	}

	public CustomVideoView(Context context, String filePath) {
		super(context);

		this.filePath = filePath;

		init();

	}

	public CustomVideoView(Context context, Uri uri) {
		super(context);

		this.uri = uri;

		init();

	}

	private void init() {

		setEGLContextClientVersion(2);

		getHolder().setFormat(PixelFormat.TRANSLUCENT);
		setEGLConfigChooser(8, 8, 8, 8, 16, 0);

		mRenderer = new VideoRender(getContext());
		setRenderer(mRenderer);

	}

	@Override
	public void onResume() {
		super.onResume();
	}

	@Override
	public void onPause() {
		super.onPause();
	}

	@Override
	protected void onDetachedFromWindow() {
		// TODO Auto-generated method stub
		super.onDetachedFromWindow();

		if (mMediaPlayer != null) {
			mMediaPlayer.stop();
			mMediaPlayer.release();
		}
	}

	private class VideoRender implements GLSurfaceView.Renderer,
	SurfaceTexture.OnFrameAvailableListener {
		private String TAG = "VideoRender";

		private static final int FLOAT_SIZE_BYTES = 4;
		private static final int TRIANGLE_VERTICES_DATA_STRIDE_BYTES = 3 * FLOAT_SIZE_BYTES;
		private static final int TEXTURE_VERTICES_DATA_STRIDE_BYTES = 2 * FLOAT_SIZE_BYTES;
		private static final int TRIANGLE_VERTICES_DATA_POS_OFFSET = 0;
		private static final int TRIANGLE_VERTICES_DATA_UV_OFFSET = 0;
		private final float[] mTriangleVerticesData = { -1.0f, -1.0f, 0, 1.0f,
				-1.0f, 0, -1.0f, 1.0f, 0, 1.0f, 1.0f, 0, };

		private final float[] mTextureVerticesData = { 0.f, 0.0f, 1.0f, 0.f,
				0.0f, 1.f, 1.0f, 1.0f };

		private FloatBuffer mTriangleVertices;

		// extra
		private FloatBuffer mTextureVertices;

		private final String mVertexShader = "uniform mat4 uMVPMatrix;\n"
				+ "uniform mat4 uSTMatrix;\n" + "attribute vec4 aPosition;\n"
				+ "attribute vec4 aTextureCoord;\n"
				+ "varying vec2 vTextureCoord;\n" + "void main() {\n"
				+ "  gl_Position = uMVPMatrix * aPosition;\n"
				+ "  vTextureCoord = (uSTMatrix * aTextureCoord).xy;\n" + "}\n";

		private final String mFragmentShader = "#extension GL_OES_EGL_image_external : require\n"
				+ "precision mediump float;\n"
				+ "varying vec2 vTextureCoord;\n"
				+ "uniform samplerExternalOES sTexture;\n"
				+ "void main() {\n"
				+ "  gl_FragColor = texture2D(sTexture, vTextureCoord);\n"
				+ "}\n";

		private float[] mMVPMatrix = new float[16];
		private float[] mSTMatrix = new float[16];
		private float[] projectionMatrix = new float[16];

		private int mProgram;
		private int mTextureID;
		private int muMVPMatrixHandle;
		private int muSTMatrixHandle;
		private int maPositionHandle;
		private int maTextureHandle;

		private SurfaceTexture mSurface;
		private boolean updateSurface = false;

		private int GL_TEXTURE_EXTERNAL_OES = 0x8D65;

		public VideoRender(Context context) {
			mTriangleVertices = ByteBuffer
					.allocateDirect(
							mTriangleVerticesData.length * FLOAT_SIZE_BYTES)
							.order(ByteOrder.nativeOrder()).asFloatBuffer();
			mTriangleVertices.put(mTriangleVerticesData).position(0);

			// extra
			mTextureVertices = ByteBuffer
					.allocateDirect(
							mTextureVerticesData.length * FLOAT_SIZE_BYTES)
							.order(ByteOrder.nativeOrder()).asFloatBuffer();
			mTextureVertices.put(mTextureVerticesData).position(0);

			Matrix.setIdentityM(mSTMatrix, 0);
		}

		public void onDrawFrame(GL10 glUnused) {

			synchronized (this) {
				if (updateSurface) {
					mSurface.updateTexImage();
					mSurface.getTransformMatrix(mSTMatrix);
					updateSurface = false;
				} else {
					return;
				}
			}

			GLES20.glClearColor(255.0f, 255.0f, 255.0f, 1.0f);
			GLES20.glClear(GLES20.GL_DEPTH_BUFFER_BIT
					| GLES20.GL_COLOR_BUFFER_BIT);

			GLES20.glUseProgram(mProgram);
			checkGlError("glUseProgram");

			GLES20.glActiveTexture(GLES20.GL_TEXTURE0);
			GLES20.glBindTexture(GL_TEXTURE_EXTERNAL_OES, mTextureID);

			mTriangleVertices.position(TRIANGLE_VERTICES_DATA_POS_OFFSET);
			GLES20.glVertexAttribPointer(maPositionHandle, 3, GLES20.GL_FLOAT,
					false, TRIANGLE_VERTICES_DATA_STRIDE_BYTES,
					mTriangleVertices);
			checkGlError("glVertexAttribPointer maPosition");
			GLES20.glEnableVertexAttribArray(maPositionHandle);
			checkGlError("glEnableVertexAttribArray maPositionHandle");

			mTextureVertices.position(TRIANGLE_VERTICES_DATA_UV_OFFSET);
			GLES20.glVertexAttribPointer(maTextureHandle, 2, GLES20.GL_FLOAT,
					false, TEXTURE_VERTICES_DATA_STRIDE_BYTES, mTextureVertices);

			checkGlError("glVertexAttribPointer maTextureHandle");
			GLES20.glEnableVertexAttribArray(maTextureHandle);
			checkGlError("glEnableVertexAttribArray maTextureHandle");

			Matrix.setIdentityM(mMVPMatrix, 0);

			GLES20.glUniformMatrix4fv(muMVPMatrixHandle, 1, false, mMVPMatrix,
					0);
			GLES20.glUniformMatrix4fv(muSTMatrixHandle, 1, false, mSTMatrix, 0);

			GLES20.glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, 4);
			checkGlError("glDrawArrays");
			GLES20.glFinish();

		}

		public void onSurfaceChanged(GL10 glUnused, int width, int height) {

			GLES20.glViewport(0, 0, width, height);

			Matrix.frustumM(projectionMatrix, 0, -1.0f, 1.0f, -1.0f, 1.0f,
					1.0f, 10.0f);

		}

		@Override
		public void onSurfaceCreated(GL10 gl, EGLConfig config) {
			mProgram = createProgram(mVertexShader, mFragmentShader);
			if (mProgram == 0) {
				return;
			}
			maPositionHandle = GLES20
					.glGetAttribLocation(mProgram, "aPosition");
			checkGlError("glGetAttribLocation aPosition");
			if (maPositionHandle == -1) {
				throw new RuntimeException(
						"Could not get attrib location for aPosition");
			}
			maTextureHandle = GLES20.glGetAttribLocation(mProgram,
					"aTextureCoord");
			checkGlError("glGetAttribLocation aTextureCoord");
			if (maTextureHandle == -1) {
				throw new RuntimeException(
						"Could not get attrib location for aTextureCoord");
			}

			muMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram,
					"uMVPMatrix");
			checkGlError("glGetUniformLocation uMVPMatrix");
			if (muMVPMatrixHandle == -1) {
				throw new RuntimeException(
						"Could not get attrib location for uMVPMatrix");
			}

			muSTMatrixHandle = GLES20.glGetUniformLocation(mProgram,
					"uSTMatrix");
			checkGlError("glGetUniformLocation uSTMatrix");
			if (muSTMatrixHandle == -1) {
				throw new RuntimeException(
						"Could not get attrib location for uSTMatrix");
			}

			int[] textures = new int[1];
			GLES20.glGenTextures(1, textures, 0);

			mTextureID = textures[0];
			GLES20.glBindTexture(GL_TEXTURE_EXTERNAL_OES, mTextureID);
			checkGlError("glBindTexture mTextureID");

			GLES20.glTexParameterf(GL_TEXTURE_EXTERNAL_OES,
					GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_NEAREST);
			GLES20.glTexParameterf(GL_TEXTURE_EXTERNAL_OES,
					GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR);

			mSurface = new SurfaceTexture(mTextureID);
			mSurface.setOnFrameAvailableListener(this);

			Surface surface = new Surface(mSurface);

			mMediaPlayer = new MediaPlayer();

			if (file != null) {
				try {
					mMediaPlayer.setDataSource(file.getAbsolutePath());
				} catch (IllegalArgumentException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (SecurityException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IllegalStateException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			} else if (filePath != null) {
				try {
					mMediaPlayer.setDataSource(filePath);
				} catch (IllegalArgumentException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (SecurityException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IllegalStateException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			} else if (uri != null) {
				try {
					mMediaPlayer.setDataSource(getContext(), uri);
				} catch (IllegalArgumentException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (SecurityException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IllegalStateException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}

			mMediaPlayer.setSurface(surface);
			surface.release();

			try {
				mMediaPlayer.prepare();
			} catch (IOException t) {
				Log.e(TAG, "media player prepare failed");
			}

			synchronized (this) {
				updateSurface = false;
			}

			mMediaPlayer.start();

		}

		synchronized public void onFrameAvailable(SurfaceTexture surface) {

			updateSurface = true;
		}

		private int loadShader(int shaderType, String source) {
			int shader = GLES20.glCreateShader(shaderType);
			if (shader != 0) {
				GLES20.glShaderSource(shader, source);
				GLES20.glCompileShader(shader);
				int[] compiled = new int[1];
				GLES20.glGetShaderiv(shader, GLES20.GL_COMPILE_STATUS,
						compiled, 0);
				if (compiled[0] == 0) {
					Log.e(TAG, "Could not compile shader " + shaderType + ":");
					Log.e(TAG, GLES20.glGetShaderInfoLog(shader));
					GLES20.glDeleteShader(shader);
					shader = 0;
				}
			}
			return shader;
		}

		private int createProgram(String vertexSource, String fragmentSource) {
			int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, vertexSource);
			if (vertexShader == 0) {
				return 0;
			}
			int pixelShader = loadShader(GLES20.GL_FRAGMENT_SHADER,
					fragmentSource);
			if (pixelShader == 0) {
				return 0;
			}

			int program = GLES20.glCreateProgram();
			if (program != 0) {
				GLES20.glAttachShader(program, vertexShader);
				checkGlError("glAttachShader");
				GLES20.glAttachShader(program, pixelShader);
				checkGlError("glAttachShader");
				GLES20.glLinkProgram(program);
				int[] linkStatus = new int[1];
				GLES20.glGetProgramiv(program, GLES20.GL_LINK_STATUS,
						linkStatus, 0);
				if (linkStatus[0] != GLES20.GL_TRUE) {
					Log.e(TAG, "Could not link program: ");
					Log.e(TAG, GLES20.glGetProgramInfoLog(program));
					GLES20.glDeleteProgram(program);
					program = 0;
				}
			}
			return program;
		}

		private void checkGlError(String op) {
			int error;
			while ((error = GLES20.glGetError()) != GLES20.GL_NO_ERROR) {
				Log.e(TAG, op + ": glError " + error);
				throw new RuntimeException(op + ": glError " + error);
			}
		}

	}

}
