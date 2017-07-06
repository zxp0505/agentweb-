## AgentWeb源码分析 ##
https://github.com/Justson/AgentWeb

使用方式

 				mAgentWeb = AgentWeb.with(this)//
                .setAgentWebParent(mLinearLayout,new LinearLayout.LayoutParams(-1,-1) )//
                .useDefaultIndicator()//使用默认进度条
                .defaultProgressBarColor()//使用默认进度条颜色
                .setReceivedTitleCallback(mCallback) //设置web界面title的回掉
                .setWebChromeClient(mWebChromeClient)//进度显示
                .setWebViewClient(mWebViewClient)//
                .setSecutityType(AgentWeb.SecurityType.strict)//暂时未知
                .setWebLayout(new WebLayout(this))//添加web的布局  里面包含webview
                .createAgentWeb()//
                .ready()
                .go(getUrl());

        mAgentWeb.getLoader().loadUrl(getUrl());

Agentweb  
AgentBuilder

 		private Activity mActivity;
        private ViewGroup mViewGroup;
        private boolean isNeedProgress;
        private int index = -1;
        private BaseIndicatorView v;
        private IndicatorController mIndicatorController = null;
        /*默认进度条是打开的*/
        private boolean enableProgress = true;
        private ViewGroup.LayoutParams mLayoutParams = null;
        private WebViewClient mWebViewClient;
        private WebChromeClient mWebChromeClient;
        private int mIndicatorColor = -1; //进度条颜色
        private WebSettings mWebSettings;
        private WebCreator mWebCreator;
        private WebViewClientCallbackManager   mWebViewClientCallbackManager = new WebViewClientCallbackManager();   //回调的管理者   ReceivedTitleCallback（包含title变化的回调管理） AgentWebCompatInterface
 
        private SecurityType mSecurityType = SecurityType.default_check;

        private ChromeClientCallbackManager mChromeClientCallbackManager = new ChromeClientCallbackManager();

        private Map<String, String> headers = null;


        private ArrayMap<String, Object> mJavaObject = null;
        private int mIndicatorColorWithHeight = -1;
        private WebView mWebView;
        private boolean webclientHelper =true;
        public ArrayList<DownLoadResultListener> mDownLoadResultListeners;
        public IWebLayout mWebLayout;//web的布局  里面包含webview

IndicatorBuilder  进度条

关键点：

.createAgentWeb()方法


 		public PreAgentWeb createAgentWeb() {
            return mAgentBuilder.buildAgentWeb();
        }

		private PreAgentWeb buildAgentWeb() {
            return new PreAgentWeb(HookManager.hookAgentWeb(new AgentWeb(this), this));
        }

在new Agentweb(this)这个构造方法中  初始化很多信息 目的是将Agentbuild的内部的数据 赋值与 AgentWeb中  

	 private AgentWeb(AgentBuilder agentBuilder) {
        this.mActivity = agentBuilder.mActivity;
        this.mViewGroup = agentBuilder.mViewGroup;
        this.enableProgress = agentBuilder.enableProgress;
		
        mWebCreator = agentBuilder.mWebCreator == null ? configWebCreator(agentBuilder.v, agentBuilder.index, agentBuilder.mLayoutParams, agentBuilder.mIndicatorColor, agentBuilder.mIndicatorColorWithHeight, agentBuilder.mWebView,agentBuilder.mWebLayout) : 
		agentBuilder.mWebCreator;
		//主要是为了构造 DefaultWebCreator  在这个类，下面确定webview 已经顶部进度条的配置
        mIndicatorController = agentBuilder.mIndicatorController;
        this.mWebChromeClient = agentBuilder.mWebChromeClient;
        this.mWebViewClient = agentBuilder.mWebViewClient;
        mAgentWeb = this;
        this.mWebSettings = agentBuilder.mWebSettings;
        this.mIEventHandler = agentBuilder.mIEventHandler;
        TAG_TARGET = ACTIVITY_TAG;
        if (agentBuilder.mJavaObject != null && agentBuilder.mJavaObject.isEmpty())
            this.mJavaObjects.putAll((Map<? extends String, ?>) agentBuilder.mJavaObject);
        this.mChromeClientCallbackManager = agentBuilder.mChromeClientCallbackManager;
        this.mWebViewClientCallbackManager = agentBuilder.mWebViewClientCallbackManager;

        this.mSecurityType = agentBuilder.mSecurityType;
		//查看下面loader类分析
        this.mILoader = new LoaderImpl(mWebCreator.create().get(), agentBuilder.headers);
		//生命周期控制类 查看下面解析
        this.mWebLifeCycle = new DefaultWebLifeCycleImpl(mWebCreator.get());
        mWebSecurityController = new WebSecurityControllerImpl(mWebCreator.get(), this.mAgentWeb.mJavaObjects, mSecurityType);
        this.webClientHelper=agentBuilder.webclientHelper;
		//设置下载监听
        setLoadListener(agentBuilder.mDownLoadResultListeners);
        doCompat();
        doSafeCheck();
    }

WebCreator mWebCreator;--->  DefaultWebCreator里面包含这些信息

	private Activity mActivity;
    private ViewGroup mViewGroup;
    private boolean isNeedDefaultProgress;
    private int index;
    private BaseIndicatorView progressView;
    private ViewGroup.LayoutParams mLayoutParams = null;
    private int color = -1;
    private int height_dp;
    private boolean isCreated=false;
    private IWebLayout mIWebLayout;
    private BaseProgressSpec mBaseProgressSpec;
public DefaultWebCreator create() {


        if(isCreated){
            return this;
        }
        isCreated=true;
        ViewGroup mViewGroup = this.mViewGroup;
        if (mViewGroup == null) {
            mViewGroup = this.mFrameLayout= (FrameLayout)createGroupWithWeb();
            mActivity.setContentView(mViewGroup);
        } else {
            if (index == -1)
                mViewGroup.addView( this.mFrameLayout= (FrameLayout)createGroupWithWeb(), mLayoutParams);
            else
                mViewGroup.addView(this.mFrameLayout= (FrameLayout)createGroupWithWeb(), index, mLayoutParams);
        }
        return this;
    }

  		private ViewGroup createGroupWithWeb() {
        Activity mActivity = this.mActivity;
        FrameLayout mFrameLayout = new FrameLayout(mActivity);
        mFrameLayout.setBackgroundColor(Color.WHITE);
        View target=mIWebLayout==null?(this.mWebView= (WebView) web()):webLayout();
        FrameLayout.LayoutParams mLayoutParams = new FrameLayout.LayoutParams(-1, -1);
        mFrameLayout.addView(target, mLayoutParams);

        LogUtils.i("Info", "    webView:" + (this.mWebView instanceof AgentWebView));
        if (isNeedDefaultProgress) {
            FrameLayout.LayoutParams lp = null;
            WebProgress mWebProgress = new WebProgress(mActivity);
            if (height_dp > 0)
                lp = new FrameLayout.LayoutParams(-2, AgentWebUtils.dp2px(mActivity, height_dp));
            else
                lp = mWebProgress.offerLayoutParams();
            if (color != -1)
                mWebProgress.setColor(color);
            lp.gravity = Gravity.TOP;
            mFrameLayout.addView((View) (this.mBaseProgressSpec = mWebProgress), lp);
        } else if (!isNeedDefaultProgress && progressView != null) {
            mFrameLayout.addView((View) (this.mBaseProgressSpec = (BaseProgressSpec) progressView), progressView.offerLayoutParams());
        }
        return mFrameLayout;

    }

	 private View webLayout(){
        WebView mWebView = null;
        if((mWebView=mIWebLayout.getWeb())==null){
            mWebView=web();
            mIWebLayout.getLayout().addView(mWebView,-1,-1);
            LogUtils.i("Info","add webview");

        }else{
            AgentWebConfig.WEBVIEW_TYPE=AgentWebConfig.WEBVIEW_CUSTOM_TYPE;
        }
        this.mWebView=mWebView;
        return mIWebLayout.getLayout();

    }

	用weblayout这个类 主要用来放置webview


PreAgentWeb包含这两个信息
 		public static class PreAgentWeb {
        private AgentWeb mAgentWeb;
        private boolean isReady = false;


	public interface ILoader {


    void loadUrl(String url);

    void reload();

    void loadData(String data, String mimeType, String encoding);

    void stopLoading();

    void loadDataWithBaseURL(String baseUrl, String data,
                             String mimeType, String encoding, String historyUrl);

	}
	
	//实现ILoader  主要功能：实现任意线程安全加载url

	public class LoaderImpl implements ILoader {


    private Handler mHandler = null;
    private WebView mWebView;

    private Map<String, String> headers = null;

    LoaderImpl(WebView webView, Map<String, String> map) {
        this.mWebView = webView;
        if (this.mWebView == null)
            new NullPointerException("webview is null");

        this.headers = map;
        mHandler = new Handler(Looper.getMainLooper());
    }

生命周期控制类

	public interface WebLifeCycle {
    void onResume();
    void onPause();
    void onDestroy();
	}
	public class DefaultWebLifeCycleImpl implements WebLifeCycle {
    private WebView mWebView;

    DefaultWebLifeCycleImpl(WebView webView) {
        this.mWebView = webView;
    }

    @Override
    public void onResume() {
        if (this.mWebView != null) {

            if (Build.VERSION.SDK_INT >= 11)
                this.mWebView.onResume();

            this.mWebView.resumeTimers();
        }


    }

    @Override
    public void onPause() {

        if (this.mWebView != null) {
            this.mWebView.pauseTimers();
            if (Build.VERSION.SDK_INT >= 11)
                this.mWebView.onPause();
        }
    }

    @Override
    public void onDestroy() {

        AgentWebUtils.clearWebView(this.mWebView);
    }

	public static final void clearWebView(WebView m) {

        if (m == null)
            return;
        if (Looper.myLooper() != Looper.getMainLooper())
            return;
        m.loadUrl("about:blank");
        m.stopLoading();
        if (m.getHandler() != null)
            m.getHandler().removeCallbacksAndMessages(null);
        m.removeAllViews();
        ViewGroup mViewGroup = null;
        if ((mViewGroup = ((ViewGroup) m.getParent())) != null)
            mViewGroup.removeView(m);
        m.setWebChromeClient(null);
        m.setWebViewClient(null);
        m.setTag(null);
        m.clearHistory();
        m.destroy();
        m = null;


    }


	 private AgentWeb ready() {

        AgentWebConfig.initCookiesManager(mActivity.getApplicationContext());
        WebSettings mWebSettings = this.mWebSettings;
        if (mWebSettings == null) {
            this.mWebSettings = mWebSettings = WebDefaultSettingsManager.getInstance();
        }
        if (mWebListenerManager == null && mWebSettings instanceof WebDefaultSettingsManager) {
            mWebListenerManager = (WebListenerManager) mWebSettings;
        }
        mWebSettings.toSetting(mWebCreator.get());
		//js注解 安全检查 以及注解检查
        if (mJsInterfaceHolder == null) {
			//添加js调用anroid的接口声明
            mJsInterfaceHolder = JsInterfaceHolderImpl.getJsInterfaceHolder(mWebCreator.get(), this.mSecurityType);
        }
        if (mJavaObjects != null && !mJavaObjects.isEmpty()) {
            mJsInterfaceHolder.addJavaObjects(mJavaObjects);
        }
		//设置下载监听
        mWebListenerManager.setDownLoader(mWebCreator.get(), getLoadListener());
        //
		mWebListenerManager.setWebChromeClient(mWebCreator.get(), getChromeClient());
        mWebListenerManager.setWebViewClient(mWebCreator.get(), getClient());


        return this;
    }





查看 js与webview交互 
http://blog.csdn.net/carson_ho/article/details/64904691 
http://blog.csdn.net/carson_ho/article/details/64904635 //漏洞分析 以及解决方案

js与webview交互有三种方法  ：

 ![](http://upload-images.jianshu.io/upload_images/944365-8c91481325a5253e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


自己以前封装的简易webview https://github.com/zxp0505/MyWebview