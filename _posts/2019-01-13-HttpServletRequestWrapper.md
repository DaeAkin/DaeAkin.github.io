## HttpServletRequestWrapper

요즘 웹 애플리케이션에서는 보안을 위한 다양한 방법이 존재한다. 어떠한 요청을 처리하기 전에 이 사용자가 권한을 가지고 있는지 확인하기 위해 해당 메소드가 호출되기전에 권한 인증을 하게 된다. 나는 JWT를 이용해서 token 인증 방식을 이용했는데, 메소드를 호출하기전에 token을 인증하는 방식이다. 클라이언트는 헤더나 request 객체에 담아 token의 값을 서버로 보낸다.
	하지만 문제가 있는데 넘어온 request 객체는 InputStream에 의해 읽혀진다. 인증 과정에서 request를 읽어 버리면 이 request는 더 이상 읽을 수가 없게 된다. 톰캣에서 막아놨기 때문이다. request를 읽지 못하여 인증에 성공하더라도 그 이후 과정에서(Controller가 request를 다시 읽는 다던지) 다시 사용하지 못한다. 
	이 방법을 해결하기 위해서는 `javax.servlet.http`에 있는 `HttpServletRequestWrapper`를 이용해서 wrapper클래스를 만들면 된다.



> JsonRequestWrapper

```java
import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Map;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

import com.fasterxml.jackson.core.JsonGenerationException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.ObjectMapper;

public class JsonRequestWrapper extends HttpServletRequestWrapper {

	 private final String body;
	 
	    public JsonRequestWrapper(HttpServletRequest request) throws IOException
	    {
	        //So that other request method behave just like before
	        super(request);
	         
	        StringBuilder stringBuilder = new StringBuilder();
	        BufferedReader bufferedReader = null;
	        try {
	            InputStream inputStream = request.getInputStream();
	            if (inputStream != null) {
	                bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
	                char[] charBuffer = new char[128];
	                int bytesRead = -1;
	                while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
	                    stringBuilder.append(charBuffer, 0, bytesRead);
	                }
	            } else {
	                stringBuilder.append("");
	            }
	        } catch (IOException ex) {
	            throw ex;
	        } finally {
	            if (bufferedReader != null) {
	                try {
	                    bufferedReader.close();
	                } catch (IOException ex) {
	                    throw ex;
	                }
	            }
	        }
	        //Store request pody content in 'body' variable
	        body = stringBuilder.toString();
	    }
	 
	    @Override
	    public ServletInputStream getInputStream() throws IOException {
	        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(body.getBytes());
	        ServletInputStream servletInputStream = new ServletInputStream() {
	            public int read() throws IOException {
	                return byteArrayInputStream.read();
	            }
	        };
	        return servletInputStream;
	    }
	 
	    @Override
	    public BufferedReader getReader() throws IOException {
	        return new BufferedReader(new InputStreamReader(this.getInputStream()));
	    }
	 
	    //Use this method to read the request body N times
	    public String getBody() {
	        return this.body;
	    }

	public Map<String, Object> jsonToMap() {
		Map<String, Object> map = null;
		try {

			ObjectMapper mapper = new ObjectMapper();

			// read JSON from a file
			 map = mapper.readValue(
					body, 
					new TypeReference<Map<String, Object>>() {
			});



		} catch (JsonGenerationException e) {
			e.printStackTrace();
		} catch (JsonMappingException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	

		return map;
	}

}

```



나는 filter를 이용해서 token 인증을 구현했다.

> JWTFilter

```java
import java.io.IOException;
import java.util.Map;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

public class JWTFilter implements Filter{
	
	@Override
	public void init(FilterConfig config) throws ServletException {
		System.out.println("---- init call ----");
		
	}
	
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		

//		 Wrapper 생성.
		JsonRequestWrapper jrw = new JsonRequestWrapper((HttpServletRequest)request);
			
		System.out.println("---- doFilter ----");
		
        //필자는 header를 이용했다.
		String loginToken = jrw.getHeader("token");
		
		JWTUtil.verifyToken(loginToken);
			
		// getbody()를 이용해서 wrapper에 담긴 값들을 가져온다.	
		jrw.getbody();
		
        // 그 후에 request를 이용해야할 메소드들에게 wrapper에 담긴내용들을 전달해준다.
		chain.doFilter(jrw, response);
	}

	@Override
	public void destroy() {
		// TODO Auto-generated method stub
		
	}


}

```

