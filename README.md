# Retrofit-Interceptor
##Retrofit 利用Interceptor(拦截器) 添加请求的固定参数,加密参数,可变参数
有一个情景:联网框架早已搭好,项目已经进行了好几期,领导突然说要在所有请求上加一个公共参数,原来的请求中有一个签名参数,是根据其他所有参数算出来                            的一个字符串,添加新参数,原来的签名参数也要重新算,接口上一个一个加?你在逗我么......<br>
于是我对拦截器进行了一下研究<br>
因为我的项目是迭代开发所以已经写完的没有改,新建的项目直接把公共参数添加在request.url().url()的后面重新拼上去即可<br>
以下是代码<br>
```Java
/**
 * @author: gh
 * @description: 网络请求拦截器
 * @date: 2017/2/14 09:31
 * @note:
 */

public class NetInterceptor implements Interceptor {

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();

        Map<String, Object> map = new HashMap();
        //得到？后面的部分
        String url_param = request.url().url().toString().split("[?]")[1];
        //把每个参数分开
        String[] params = url_param.split("&");
        //分开键和值
        for (String param : params) {
            String[] keyvalue = param.split("=");
            keyvalue[1] = URLDecoder.decode(keyvalue[1],"UTF-8");
            if (!"sign".equals(keyvalue[0]))
                map.put(keyvalue[0], keyvalue[1]);
        }
        //添加新参数
//        map.put("version", "11");
        //签名
        String sign = null;
        try {
            sign = SignUtil.sign(map, RXApiConstants.CLIENT_SECRET_VALUE);
        } catch (SignatureException e) {
            e.printStackTrace();
        }
        //重新定义url
        //拼参数
        List<String> paramNames = new ArrayList(map.size());
        paramNames.addAll(map.keySet());
        StringBuilder sb = new StringBuilder();
        for (String paramName : paramNames) {
            sb.append(paramName).append("=").append(URLEncoder.encode((String) map.get(paramName), "UTF-8")).append("&");
        }
        //拼签名
        sb.append("sign=" ).append(sign);
        //最终的url
        String newUrl = request.url().url().toString().split("[?]")[0]+"?"+sb.toString();

        Request.Builder builder = request.newBuilder()
                .url(newUrl);
        Request request2 = builder.build();
        return chain.proceed(request2);
    }
}
