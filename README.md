<h1>서블릿</h1>
1. @WebServlet 서블릿 애노테이션
- name: 서블릿 이름
- urlPatterns: URL 매핑
EX) @WebServlet(name = "helloServlet", urlPatterns = "/hello")

2. HTTP 요청 메시지 로그로 확인하기
  1) appliaction.properties에
  2) logging.level.org.apache.coyote.http11=debug 추가
     
3. 웹 어플리케이션 서버의 요청 응답 구
![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/8147ba74-73a0-46aa-8d20-e8fdc68fc7ec)

4. HttpServletRequest 역할
HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 매우 불편할 것이다.
서블릿은 개발자가 HTTP 요청메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다.
그리고 그 결과를 HttpServletRequest 객체에 담아서 제공한다

5. HTTP 요청 데이터 - 개요
   1) GET
      - 쿼리 파라미터 - /url**?username=hello&age=20**\
      - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
      - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
   2) POST - HTML Form
      - content-type: application/x-www-form-urlencoded
      - 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
      - 예) 회원 가입, 상품 주문, HTML Form 사용
      - HTTP message body에 데이터를 직접 담아서 요청
   3) HTTP API에서 주로 사용, JSON, XML, TEXT
      - 데이터 형식은 주로 JSON 사용
      - POST, PUT, PATCH
     
※ 참고
JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson 같은 JSON 변환 라이브러리를 추가해서 사용해야 한다. 
스프링 부트로 Spring MVC를 선택하면 기본으로 Jackson 라이브러리 ( ObjectMapper )를 함께 제공한다.

※  참고
HTML form 데이터도 메시지 바디를 통해 전송되므로 직접 읽을 수 있다. 
하지만 편리한 파리미터 조회 기능( request.getParameter(...) )을 이미 제공하기 때문에 파라미터 조회 기능을 사용하면 된다.

6. HttpServletResponse
1) HTTP 응답 메시지 생성
   - HTTP 응답코드 지정
   - 헤더 생성
   - 바디 생성
  (1) 편의 기능 제공
  - Content-Type, 쿠키, Redirect

7. HTTP 응답 데이터 - API JSON

```
package hello.servlet.basic.response;
import com.fasterxml.jackson.databind.ObjectMapper;
import hello.servlet.basic.HelloData;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
/**
 * http://localhost:8080/response-json
 *
 */
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {
     private ObjectMapper objectMapper = new ObjectMapper();

      @Override
     protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
     //Content-Type: application/json
     response.setHeader("content-type", "application/json");
     response.setCharacterEncoding("utf-8");

     HelloData data = new HelloData();
     data.setUsername("kim");
     data.setAge(20);

     //{"username":"kim","age":20}
     String result = objectMapper.writeValueAsString(data);
     response.getWriter().write(result);
   }
}
```

- HTTP 응답으로 JSON을 반환할 때는 content-type을 application/json 로 지정해야 한다.
- Jackson 라이브러리가 제공하는 objectMapper.writeValueAsString() 를 사용하면 객체를 JSON 문자로
변경할 수 있다.

※ 참고
- application/json 은 스펙상 utf-8 형식을 사용하도록 정의되어 있다. 그래서 스펙에서 charset=utf-8 과 같은 추가 파라미터를 지원하지 않는다. 따라서 application/json 이라고만 사용해야지
application/json;charset=utf-8 이라고 전달하는 것은 의미 없는 파라미터를 추가한 것이 된다.
response.getWriter()를 사용하면 추가 파라미터를 자동으로 추가해버린다. 이때는 response.getOutputStream()으로 출력하면 그런 문제가 없다

<h1>서블릿, JSP, MVC 패턴</h1>
```
package hello.servlet.domain.member;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
/**
 * 동시성 문제가 고려되어 있지 않음, 실무에서는 ConcurrentHashMap, AtomicLong 사용 고려
 */
public class MemberRepository {
     private static Map<Long, Member> store = new HashMap<>(); //static 사용
       
     private static long sequence = 0L; //static 사용
       
     private static final MemberRepository instance = new MemberRepository();
     
     public static MemberRepository getInstance() {
       return instance;
     }
     
     private MemberRepository() {
     
     }
     
     public Member save(Member member) {
       member.setId(++sequence);
       store.put(member.getId(), member);
       return member;
     }
     
     public Member findById(Long id) {
       return store.get(id);
     }
     
     public List<Member> findAll() {
       return new ArrayList<>(store.values());
     }
     
     public void clearStore() {
       store.clear();
     }
}
```

- 회원 저장소는 싱글톤 패턴을 적용했다. 스프링을 사용하면 스프링 빈으로 등록하면 되지만, 지금은 최대한 스프링 없이
순수 서블릿 만으로 구현하는 것이 목적이다.
- 싱글톤 패턴은 객체를 단 하나만 생생해서 공유해야 하므로 생성자를 private 접근자로 막아둔다.

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/c9cbe5ab-6bf2-42dd-9950-1584d1403bde)


※ 참고
redirect vs forward
리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다. 
따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.


<h3>MVC 패턴 - 한계</h3>
MVC 패턴을 적용한 덕분에 컨트롤러의 역할과 뷰를 렌더링 하는 역할을 명확하게 구분할 수 있다.
특히 뷰는 화면을 그리는 역할에 충실한 덕분에, 코드가 깔끔하고 직관적이다. 단순하게 모델에서 필요한 데이터를 꺼내고, 화면을 만들면 된다.
그런데 컨트롤러는 딱 봐도 중복이 많고, 필요하지 않는 코드들도 많이 보인다.

<h4>MVC 컨트롤러의 단점</h4>
1. 포워드 중복
View로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이 부분을 메서드로 공통화해도 되지만, 해당 메서드도 항상 직접 호출해야 한다
```
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

2.ViewPath에 중복
```
String viewPath = "/WEB-INF/views/new-form.jsp";
```

3. 사용하지 않는 코드
다음 코드를 사용할 때도 있고, 사용하지 않을 때도 있다. 특히 response는 현재 코드에서 사용되지 않는다. ```
```
HttpServletRequest request, HttpServletResponse response
```
그리고 이런 HttpServletRequest , HttpServletResponse 를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.

4. 공통 처리가 어렵다.
기능이 복잡해질 수 록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 될 것이다. 그리고 호출하는 것 자체도 중복이다.
정리하면 공통 처리가 어렵다는 문제가 있다.
이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 역할을 하는 기능이 필요하다.  프론트 컨트롤러(Front Controller) 패턴을 도입하면 이런 문제를 깔끔하게 해결할 수 있다.(입구를 하나로!)
스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다

<h1>MVC 프레임워크 만들기</h1>

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/a3d5d4d3-a763-41b7-89e8-48c23b9fdc57)

<h3>FrontController 패턴 특징</h3>
1. 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
2. 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출
3. 공통 처리 가능
4. 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

<h3>스프링 웹 MVC와 프론트 컨트롤러</h3>
1. 스프링 웹 MVC의 핵심도 바로 FrontController
2. 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있음

```
package hello.servlet.web.frontcontroller.v1;
import hello.servlet.web.frontcontroller.v1.controller.MemberFormControllerV1;
import hello.servlet.web.frontcontroller.v1.controller.MemberListControllerV1;
import hello.servlet.web.frontcontroller.v1.controller.MemberSaveControllerV1;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {
   private Map<String, ControllerV1> controllerMap = new HashMap<>();
   public FrontControllerServletV1() {
     controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
     controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
     controllerMap.put("/front-controller/v1/members", newMemberListControllerV1());
   }
   @Override
   protected void service(HttpServletRequest request, HttpServletResponse response)  throws ServletException, IOException {
     System.out.println("FrontControllerServletV1.service");
     String requestURI = request.getRequestURI();
     ControllerV1 controller = controllerMap.get(requestURI);
     
     if (controller == null) {
       response.setStatus(HttpServletResponse.SC_NOT_FOUND);
     return;
   }
     controller.process(request, response);
   }
}
```

- urlPatterns
1) urlPatterns = "/front-controller/v1/*" : /front-controller/v1 를 포함한 하위 모든 요청은 이 서블릿에서 받아들인다.
2) 예) /front-controller/v1 , /front-controller/v1/a , /front-controller/v1/a/b

<h2>Model 추가</h2>
<h3>서블릿 종속성 제거</h3>
- 컨트롤러 입장에서 HttpServletRequest, HttpServletResponse이 꼭 필요할까?
- 요청 파라미터 정보는 자바의 Map으로 대신 넘기도록 하면 지금 구조에서는 컨트롤러가 서블릿 기술을 몰라도 동작할 수 있다.
- 그리고 request 객체를 Model로 사용하는 대신에 별도의 Model 객체를 만들어서 반환하면 된다.
- 우리가 구현하는 컨트롤러가 서블릿 기술을 전혀 사용하지 않도록 변경해보자.
 -> 이렇게 하면 구현 코드도 매우 단순해지고, 테스트 코드 작성이 쉽다.

<h3>뷰 이름 중복 제거</h3>
- 컨트롤러에서 지정하는 뷰 이름에 중복이 있는 것을 확인할 수 있다.
- 컨트롤러는 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화 하자.
-> 이렇게 해두면 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 고치면 된다

```
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {
 private Map<String, ControllerV3> controllerMap = new HashMap<>();
   public FrontControllerServletV3() {
     controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
     controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
     controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
   }
   @Override
   protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       String requestURI = request.getRequestURI();
       ControllerV3 controller = controllerMap.get(requestURI);
       
       if (controller == null) {
         response.setStatus(HttpServletResponse.SC_NOT_FOUND);
         return;
       }
      
       Map<String, String> paramMap = createParamMap(request);
       ModelView mv = controller.process(paramMap);
       String viewName = mv.getViewName();
       MyView view = viewResolver(viewName);
       view.render(mv.getModel(), request, response);
     }
     
     private Map<String, String> createParamMap(HttpServletRequest request) {
       Map<String, String> paramMap = new HashMap<>();
       request.getParameterNames().asIterator()
       .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
     }
     
     private MyView viewResolver(String viewName) {
       return new MyView("/WEB-INF/views/" + viewName + ".jsp");
     }
}
```

createParamMap()
- HttpServletRequest에서 파라미터 정보를 꺼내서 Map으로 변환한다. 그리고 해당 Map( paramMap )을 컨트롤러에 전달하면서 호출한다

<h3>뷰 리졸버</h3>
```
MyView view = viewResolver(viewName)
```
컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다. 그리고 실제 물리 경로가 있는 MyView 객체를 반환한다.

```
view.render(mv.getModel(), request, response)
```
- 뷰 객체를 통해서 HTML 화면을 렌더링 한다.
- 뷰 객체의 render() 는 모델 정보도 함께 받는다.
- JSP는 request.getAttribute() 로 데이터를 조회하기 때문에, 모델의 데이터를 꺼내서 request.setAttribute() 로 담아둔다.
- JSP로 포워드 해서 JSP를 렌더링 한다

```
public class MyView {
     private String viewPath;
     public MyView(String viewPath) {
       this.viewPath = viewPath;
     }
     public void render(HttpServletRequest request, HttpServletResponse response)  throws ServletException, IOException {
       RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
       dispatcher.forward(request, response);
     }
     public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       modelToRequestAttribute(model, request);
       RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
       dispatcher.forward(request, response);
     }
     private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
       model.forEach((key, value) -> request.setAttribute(key, value));
     }
}
```

<h3>단순하고 실용적인 컨트롤러</h3>

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/c72ebde8-1c96-4878-8dd1-8dc200492f7a)

1. 모델 객체 전달
```
Map<String, Object> model = new HashMap<>(); //추가
```

- 모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다. 컨트롤러에서 모델 객체에 값을 담으면 여기에 그대로 담겨있게 된다.
2. 뷰의 논리 이름을 직접 반환
```java
String viewName = controller.process(paramMap, model);
MyView view = viewResolver(viewName);
```
컨트롤로가 직접 뷰의 논리 이름을 반환하므로 이 값을 사용해서 실제 물리 뷰를 찾을 수 있다.

<h3>유연한 컨트롤</h3>
만약 어떤 개발자는 ControllerV3 방식으로 개발하고 싶고, 어떤 개발자는 ControllerV4 방식으로 개발하고
싶다면 어떻게 해야할까?

-> 어댑터 패턴
어댑터 패턴을 사용해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 변경해보자

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/91d32856-4474-4c5e-bccb-ace3448dccd2)

1. 핸들러 어댑터: 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다. 여기서 어댑터 역할을 해주는 덕분에 다양한 종류의 컨트롤러를 호출할 수 있다.
2. 핸들러: 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다. 그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떠한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다

```
public interface MyHandlerAdapter {
 boolean supports(Object handler);
 ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```
1. boolean supports(Object handler)
- handler는 컨트롤러를 말한다.
- 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다.
2. ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
- 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
-실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야한다.
- 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다

```
private void initHandlerMappingMap() {
   handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new
  MemberFormControllerV3());
   handlerMappingMap.put("/front-controller/v5/v3/members/save", new
  MemberSaveControllerV3());
   handlerMappingMap.put("/front-controller/v5/v3/members", new
  MemberListControllerV3());
   //V4 추가
   handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new
  MemberFormControllerV4());
   handlerMappingMap.put("/front-controller/v5/v4/members/save", new
  MemberSaveControllerV4());
   handlerMappingMap.put("/front-controller/v5/v4/members", new
  MemberListControllerV4());
}
private void initHandlerAdapters() {
   handlerAdapters.add(new ControllerV3HandlerAdapter());
   handlerAdapters.add(new ControllerV4HandlerAdapter()); //V4 추가
}
```

※ 정리
1. v1: 프론트 컨트롤러를 도입
   - 기존 구조를 최대한 유지하면서 프론트 컨트롤러를 도입
2. v2: View 분류
   - 단순 반복 되는 뷰 로직 분리
3. v3: Model 추가
   - 서블릿 종속성 제거
   - 뷰 이름 중복 제거
4. v4: 단순하고 실용적인 컨트롤러
5. v3와 거의 비슷
   - 구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 편리한 인터페이스 제공
6. v5: 유연한 컨트롤러

7. 어댑터 도입
- 어댑터를 추가해서 프레임워크를 유연하고 확장성 있게 설계
- 여기에 애노테이션을 사용해서 컨트롤러를 더 편리하게 발전시킬 수도 있다. 만약 애노테이션을 사용해서 컨트롤러를 편리하게 사용할 수 있게 하려면 어떻게 해야할까?

-> 바로 애노테이션을 지원하는 어댑터를 추가하면 된다!
-> 다형성과 어댑터 덕분에 기존 구조를 유지하면서, 프레임워크의 기능을 확장할 수 있다.

<h1>스프링 MVC - 구조 이해</h1>

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/8fdb97f3-59d0-4e25-811e-6558a1ae26e9)

<h2>DispatcherServlet 구조 살펴보기</h2>
- 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿(DispatcherServlet)

<h3>DispatcherServlet 서블릿 등록</h3>
- DispatcherServlet 도 부모 클래스에서 HttpServlet 을 상속 받아서 사용하고, 서블릿으로 동작한다.
- DispatcherServlet FrameworkServlet HttpServletBean HttpServlet
-스프링 부트는 DispatcherServlet 을 서블릿으로 자동으로 등록하면서 모든 경로( urlPatterns="/" )에 대해서 매핑한다.
※ 참고: 더 자세한 경로가 우선순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.
<h3>요청 흐름</h3>
- 서블릿이 호출되면 HttpServlet 이 제공하는 serivce() 가 호출된다.
- 스프링 MVC는 DispatcherServlet 의 부모인 FrameworkServlet 에서 service() 를 오버라이드 해두었다.
- FrameworkServlet.service() 를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch() 가 호출된다.

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;

    HandlerExecutionChain mappedHandler = null;
    ModelAndView mv = null;

    // 1. 핸들러 조회
    mappedHandler = getHandler(processedRequest);

    if (mappedHandler == null) {
      noHandlerFound(processedRequest, response);
      return;
    }
    // 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

    // 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
    // 뷰 렌더링 호출
      render(mv, request, response);
    }
    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
      View view;
      String viewName = mv.getViewName();
      // 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
      view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
      // 8. 뷰 렌더링
      view.render(mv.getModelInternal(), request, response);
}
```

<h3>동작 순서</h3>
1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다. JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.
7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다. JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

<h3>인터페이스 살펴보기</h3>
1. 스프링 MVC의 큰 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다는 점이다. 지금까지 설명한 대부분을 확장 가능할 수 있게 인터페이스로 제공한다.
2) 이 인터페이스들만 구현해서 DispatcherServlet 에 등록하면 여러분만의 컨트롤러를 만들 수도 있다.

<h1>스프링 MVC - 시작하기</h1>
1. @RequestMapping
스프링은 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로 @RequestMapping 애노테이션을 사용하는 컨트롤러이다. 다들 한번쯤 사용해보았을 것이다.

```
@Controller
public class SpringMemberFormControllerV1 {
   @RequestMapping("/springmvc/v1/members/new-form")
   public ModelAndView process() {
   return new ModelAndView("new-form");
 }
}
```

1. @Controller : 
- 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component 애노테이션이 있어서 컴포넌트 스캔의대상이 됨)
- 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
2. @RequestMapping : 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션을 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지으면 된다.
3. ModelAndView : 모델과 뷰 정보를 담아서 반환하면 된다

```
@Controller
public class SpringMemberSaveControllerV1 {
   private MemberRepository memberRepository = MemberRepository.getInstance();

   @RequestMapping("/springmvc/v1/members/save")
   public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
     String username = request.getParameter("username");
     int age = Integer.parseInt(request.getParameter("age"));
     Member member = new Member(username, age);
     System.out.println("member = " + member);
     memberRepository.save(member);
     ModelAndView mv = new ModelAndView("save-result");
     mv.addObject("member", member);
     return mv;
   }
}
```

<h5>mv.addObject("member", member)</h5>
- 스프링이 제공하는 ModelAndView 를 통해 Model 데이터를 추가할 때는 addObject() 를 사용하면된다. 
- 이 데이터는 이후 뷰를 렌더링 할 때 사용된다

<h2>스프링 MVC - 실용적인 방법</h2>
```
/**
 * v3
 * Model 도입
 * ViewName 직접 반환
 * @RequestParam 사용
 * @RequestMapping -> @GetMapping, @PostMapping
 */
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {
   private MemberRepository memberRepository = MemberRepository.getInstance();
  
   @GetMapping("/new-form")
     public String newForm() {
     return "new-form";
   }
   
   @PostMapping("/save")
   public String save(@RequestParam("username") String username, @RequestParam("age") int age,Model model) {
     Member member = new Member(username, age);
     memberRepository.save(member);
     model.addAttribute("member", member);
     return "save-result";
   }
   
   @GetMapping
   public String members(Model model) {
     List<Member> members = memberRepository.findAll();
     model.addAttribute("members", members);
     return "members";
   }
}
```
1. Model 파라미터
- save() , members() 를 보면 Model을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의 기능을 제공한다.
2. ViewName 직접 반환
- 뷰의 논리 이름을 반환할 수 있다.
3. @RequestParam 사용
- 스프링은 HTTP 요청 파라미터를 @RequestParam 으로 받을 수 있다.
- @RequestParam("username") 은 request.getParameter("username") 와 거의 같은 코드라 생각하면된다.
- 물론 GET 쿼리 파라미터, POST Form 방식을 모두 지원한다.
<h4>@RequestMapping @GetMapping, @PostMapping</h4>
- @RequestMapping 은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분할 수 있다.
- 예를 들어서 URL이 /new-form 이고, HTTP Method가 GET인 경우를 모두 만족하는 매핑을 하려면 다음과 같이
처리하면 된다.
```
@RequestMapping(value = "/new-form", method = RequestMethod.GET) 
```
이것을 @GetMapping , @PostMapping 으로 더 편리하게 사용할 수 있으며 Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다
