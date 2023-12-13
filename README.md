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


<h1>스프링 MVC - 기본 기능</h1>

※ 주의!
- Packaging는 War가 아니라 Jar를 선택해주세요. JSP를 사용하지 않기 때문에 Jar를 사용하는 것이 좋습니다. 
- 앞으로 스프링 부트를 사용하면 이 방식을 주로 사용하게 됩니다.
- Jar를 사용하면 항상 내장 서버(톰캣등)를 사용하고, webapp 경로도 사용하지 않습니다. 내장 서버 사용에 최적화 되어 있는 기능입니다. 최근에는 주로 이 방식을 사용합니다.
- War를 사용하면 내장 서버도 사용가능 하지만, 주로 외부 서버에 배포하는 목적으로 사용합니다.

<h3>로깅 간단히 알아보기</h3>

```
@RestController
public class LogTestController {
 private final Logger log = LoggerFactory.getLogger(getClass());
   @RequestMapping("/log-test")
   public String logTest() {
   String name = "Spring";
   
   log.trace("trace log={}", name);
   log.debug("debug log={}", name);
   log.info(" info log={}", name);
   log.warn(" warn log={}", name);
   log.error("error log={}", name);
   //로그를 사용하지 않아도 a+b 계산 로직이 먼저 실행됨, 이런 방식으로 사용하면 X
   log.debug("String concat log=" + name);
   return "ok";
   }
}
```

<h5>로그 선언</h5>
- private Logger log = LoggerFactory.getLogger(getClass());
- private static final Logger log = LoggerFactory.getLogger(Xxx.class)
- @Slf4j : 롬복 사용 가능

※ @RestController(Controller + ResponseBody)
- @RestController 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서
실행 결과로 ok 메세지를 받을 수 있다


<h4>테스트</h4>
1.로그가 출력되는 포멧 확인
- 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스명, 로그 메시지
2. 로그 레벨 설정을 변경해서 출력 결과를 보자.
- LEVEL: TRACE > DEBUG > INFO > WARN > ERROR
- 개발 서버는 debug 출력
- 운영 서버는 info 출력

<h4>로그 레벨 설정</h4>
```
application.properties
```

```
#전체 로그 레벨 설정(기본 info)
logging.level.root=info
#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

<h4>올바른 로그 사용법</h4>
1. log.debug("data="+data)
- 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.

2. log.debug("data={}", data)
- 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

<h4>로그 사용시 장점</h4>
1. 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
2. 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
3. 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다. 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
4. 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다

<h5>PathVariable(경로 변수) 사용</h5>

```
/**
 * PathVariable 사용
 * 변수명이 같으면 생략 가능
 * @PathVariable("userId") String userId -> @PathVariable String userId
 */
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
 log.info("mappingPath userId={}", data);
 return "ok";
}
```

- @RequestMapping 은 URL 경로를 템플릿화 할 수 있는데, @PathVariable 을 사용하면 매칭 되는 부분을
편리하게 조회할 수 있다.
- @PathVariable 의 이름과 파라미터 이름이 같으면 생략할 수 있다


<h4>PathVariable 사용 - 다중</h4>

```java
/**
 * PathVariable 사용 다중
 */
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long
orderId) {
 log.info("mappingPath userId={}, orderId={}", userId, orderId);
 return "ok";
}
```


<h4>특정 파라미터 조건 매핑</h4>
```java
/**
 * 파라미터로 추가 매핑
 * params="mode",
 * params="!mode"
 * params="mode=debug"
 * params="mode!=debug" (! = )
 * params = {"mode=debug","data=good"}
 */
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
 log.info("mappingParam");
 return "ok";
}
```

<h4>특정 헤더 조건 매핑</h4>
특정 파라미터가 있거나 없는 조건을 추가할 수 있다. 잘 사용하지는 않는다.
 ```java
/**
 * 특정 헤더로 추가 매핑
 * headers="mode",
 * headers="!mode"
 * headers="mode=debug"
 * headers="mode!=debug" (! = )
 */
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
 log.info("mappingHeader");
 return "ok";
}
```

<h4>미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume </h4>

```java
/**
 * Content-Type 헤더 기반 추가 매핑 Media Type
 * consumes="application/json"
 * consumes="!application/json"
 * consumes="application/*"
 * consumes="*\/*"
 * MediaType.APPLICATION_JSON_VALUE
 */
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
 log.info("mappingConsumes");
 return "ok";
}
```

<h4>미디어 타입 조건 매핑 - HTTP 요청 Accept, produce</h4>
```java
/**
 * Accept 헤더 기반 Media Type
 * produces = "text/html"
 * produces = "!text/html"
 * produces = "text/*"
 * produces = "*\/*"
 */
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
 log.info("mappingProduces");
 return "ok";
}
```

<h3>HTTP 요청 - 기본, 헤더 조회</h3>

```
@Slf4j
@RestController
public class RequestHeaderController {
 @RequestMapping("/headers")
 public String headers(HttpServletRequest request,
 HttpServletResponse response,
HttpMethod httpMethod,
 Locale locale,
 @RequestHeader MultiValueMap<String, String> 
headerMap,
 @RequestHeader("host") String host,
 @CookieValue(value = "myCookie", required = false) 
String cookie
 ) {
 log.info("request={}", request);
 log.info("response={}", response);
 log.info("httpMethod={}", httpMethod);
 log.info("locale={}", locale);
 log.info("headerMap={}", headerMap);
 log.info("header host={}", host);
 log.info("myCookie={}", cookie);
 return "ok";
 }
}
```

1. HttpMethod : HTTP 메서드를 조회한다. org.springframework.http.HttpMethod
2.  Locale : Locale 정보를 조회한다.
3. @RequestHeader MultiValueMap<String, String> headerMap : 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
4. @RequestHeader("host") String host : 특정 HTTP 헤더를 조회한다.
- 필수 값 여부: required
- 기본 값 속성: defaultValue
5. @CookieValue(value = "myCookie", required = false) String cookie : 특정 쿠키를 조회한다.
- 필수 값 여부: required
- 기본 값: defaultValue

<h5>MultiValueMap</h5>
1. MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
2. HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다. 
- keyA=value1&keyA=value2

```java
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");
//[value1,value2]
List<String> values = map.get("keyA")
```

<h3>HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form</h3>

클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용한다.
1. GET - 쿼리 파라미터
- /url**?username=hello&age=20**
- 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
- 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
2. POST - HTML Form
- content-type: application/x-www-form-urlencoded
- 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
- 예) 회원 가입, 상품 주문, HTML Form 사용
3. HTTP message body에 데이터를 직접 담아서 요청
- HTTP API에서 주로 사용, JSON, XML, TEXT
- 데이터 형식은 주로 JSON 사용
- POST, PUT, PATCH

<h3>HTTP 요청 파라미터 - @RequestParam</h3>

```
/**
 * @RequestParam 사용
 * - 파라미터 이름으로 바인딩
 * @ResponseBody 추가
 * - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
 */
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
 @RequestParam("username") String memberName,
 @RequestParam("age") int memberAge) {
 log.info("username={}, age={}", memberName, memberAge);
 return "ok";
}
```

- HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능

<h5>파라미터 필수 여부 - requestParamRequired</h5>

```
/**
 * @RequestParam.required
 * /request-param-required -> username이 없으므로 예외
 *
 * 주의!
 * /request-param-required?username= -> 빈문자로 통과
 *
 * 주의!
 * /request-param-required
 * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는
defaultValue 사용)
 */
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
     @RequestParam(required = true) String username,
     @RequestParam(required = false) Integer age) {
     log.info("username={}, age={}", username, age);
     return "ok";
}

```

- @RequestParam.required :파라미터 필수 여부 / 기본값이 파라미터 필수( true )이다

※ 주의! - 기본형(primitive)에 null 입력
- @RequestParam(required = false) int age
- null 을 int 에 입력하는 것은 불가능(500 예외 발생)
-> 따라서 null 을 받을 수 있는 Integer 로 변경하거나, 또는 다음에 나오는 defaultValue 사용

<h5>파라미터를 Map으로 조회하기 - requestParamMap</h5>

```
/**
 * @RequestParam Map, MultiValueMap
 * Map(key=value)
 * MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])
 */
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
 log.info("username={}, age={}", paramMap.get("username"), 
paramMap.get("age"));
 return "ok";
}
```

- 파라미터의 값이 1개가 확실하다면 Map 을 사용해도 되지만, 그렇지 않다면 MultiValueMap 을 사용하자

<h3>HTTP 요청 파라미터 - @ModelAttribute</h3>
1. HelloData 객체를 생성한다.
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.
예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다


<h2>HTTP 요청 메시지 - 단순 텍스트</h2>
<h3>HTTP message body에 데이터를 직접 담아서 요청 </h3>
1. HTTP API에서 주로 사용, JSON, XML, TEXT
2. 데이터 형식은 주로 JSON 사용
3. POST, PUT, PATCH
-요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam , 
@ModelAttribute 를 사용할 수 없다. (물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다.)

- HTTP 메시지 바디의 데이터를 InputStream 을 사용해서 직접 읽을 수 있다.

```
@Slf4j
@Controller
public class RequestBodyStringController {
 @PostMapping("/request-body-string-v1")
 public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
   ServletInputStream inputStream = request.getInputStream();
   String messageBody = StreamUtils.copyToString(inputStream, 
  StandardCharsets.UTF_8);
   log.info("messageBody={}", messageBody);
   response.getWriter().write("ok");
 }
}
````


```
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) 
throws IOException {
 String messageBody = StreamUtils.copyToString(inputStream, 
StandardCharsets.UTF_8);
 log.info("messageBody={}", messageBody);
 responseWriter.write("ok");
}
````

- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력


```
/**
 * HttpEntity: HTTP header, body 정보를 편리하게 조회
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * 응답에서도 HttpEntity 사용 가능
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
   String messageBody = httpEntity.getBody();
   log.info("messageBody={}", messageBody);
   return new HttpEntity<>("ok");
}
```

□ 스프링 MVC는 다음 파라미터를 지원한다.
1. HttpEntity
- HTTP header, body 정보를 편리하게 조회
- 메시지 바디 정보를 직접 조회
- 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
2. HttpEntity는 응답에도 사용 가능
- 메시지 바디 정보 직접 반환
- 헤더 정보 포함 가능
- view 조회X

3. HttpEntity 를 상속받은 다음 객체들도 같은 기능을 제공한다.
<h5>RequestEntity</h5>
- HttpMethod, url 정보가 추가, 요청에서 사용
<h5>ResponseEntity</h5>
- HTTP 상태 코드 설정 가능, 응답에서 사용
- return new ResponseEntity<String>("Hello World", responseHeaders, 
HttpStatus.CREATED)

※ 참고
- 스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 이때 HTTP 메시지 컨버터( HttpMessageConverter )라는 기능을 사용한다. 이것은 조금 뒤에 HTTP 메시지 컨버터에서 자세히 설명한다.

```
/**
 * @RequestBody
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * @ResponseBody
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
 log.info("messageBody={}", messageBody);
 return "ok";
}
```
<h5>@RequestBody</h5>
1. @RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam, @ModelAttribute 와는 전혀 관계가 없다.

<h5>요청 파라미터 vs HTTP 메시지 바디</h5>
- 요청 파라미터를 조회하는 기능: @RequestParam , @ModelAttribute
- HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody

<h5>@ResponseBody </h5>
- @ResponseBody 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다. 물론 이 경우에도 view를 사용하지 않는다

```
/**
 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type: 
application/json)
 *
 */
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
 log.info("username={}, age={}", data.getUsername(), data.getAge());
 return "ok";
}
```

@RequestBody 객체 파라미터
@RequestBody HelloData data
@RequestBody 에 직접 만든 객체를 지정할 수 있다.
HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문
자나 객체 등으로 변환해준다.
HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리
해준다.
자세한 내용은 뒤에 HTTP 메시지 컨버터에서 다룬다.
<h5>@RequestBody는 생략 불가능</h5>

1. 스프링은 @ModelAttribute , @RequestParam 과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용한다.
- String , int , Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)

-> 따라서 이 경우 HelloData에 @RequestBody 를 생략하면 @ModelAttribute 가 적용되어버린다.

```
HelloData data @ModelAttribute HelloData data
```

따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.

※ 주의
HTTP 요청시에 content-type이 application/json인지 꼭! 확인해야 한다. 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.

<h3>HTTP 응답 - 정적 리소스, 뷰 템플릿</h3>
- 정적 리소스
예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, 정적 리소스를 사용한다.
- 뷰 템플릿 사용
예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
- HTTP 메시지 사용
HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다

<h3>HTTP 응답 - HTTP API, 메시지 바디에 직접 입력</h3>
- HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.
- HTTP 요청에서 응답까지 대부분 다루었으므로 이번시간에는 정리를 해보자.
※ 참고
- HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달된다. 여기서 설명하는 내용은 정적 리소스나 뷰 템플릿을 거치지 않고, 직접 HTTP 응답 메시지를 전달하는 경우를 말한다

```
@Slf4j
@Controller
//@RestController
public class ResponseBodyController {
 @GetMapping("/response-body-string-v1")
 public void responseBodyV1(HttpServletResponse response) throws IOException
{
 response.getWriter().write("ok");
 }
 /**
 * HttpEntity, ResponseEntity(Http Status 추가)
 * @return
 */
 @GetMapping("/response-body-string-v2")
 public ResponseEntity<String> responseBodyV2() {
 return new ResponseEntity<>("ok", HttpStatus.OK);
 }
 @ResponseBody
 @GetMapping("/response-body-string-v3")
 public String responseBodyV3() {
 return "ok";
 }
 @GetMapping("/response-body-json-v1")
 public ResponseEntity<HelloData> responseBodyJsonV1() {
 HelloData helloData = new HelloData();
 helloData.setUsername("userA");
 helloData.setAge(20);
 return new ResponseEntity<>(helloData, HttpStatus.OK);
 }
 @ResponseStatus(HttpStatus.OK)
 @ResponseBody
 @GetMapping("/response-body-json-v2")
 public HelloData responseBodyJsonV2() {
 HelloData helloData = new HelloData();
 helloData.setUsername("userA");
 helloData.setAge(20);
 return helloData;
 }
}
```

1. responseBodyV1
- 서블릿을 직접 다룰 때 처럼 HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다.
```
response.getWriter().write("ok")
```

2. responseBodyV2
- ResponseEntity 엔티티는 HttpEntity 를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다. ResponseEntity 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
- HttpStatus.CREATED 로 변경하면 201 응답이 나가는 것을 확인할 수 있다.

3. responseBodyV3
- @ResponseBody 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다. ResponseEntity 도 동일한 방식으로 동작한다.

4.responseBodyJsonV1
- ResponseEntity 를 반환한다. HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.

5. responseBodyJsonV2
- ResponseEntity 는 HTTP 응답 코드를 설정할 수 있는데, @ResponseBody 를 사용하면 이런 것을 설정하기 까다롭다.
- @ResponseStatus(HttpStatus.OK) 애노테이션을 사용하면 응답 코드도 설정할 수 있다.

 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 동적으로 변경하려면 ResponseEntity 를 사용하면 된다.

6. @RestController
-@Controller 대신에 @RestController 애노테이션을 사용하면, 해당 컨트롤러에 모두 @ResponseBody 가
적용되는 효과가 있다.
-  따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 이름 그대로 Rest API(HTTP API)를 만들 때 사용하는 컨트롤러이다.
- 참고로 @ResponseBody 는 클래스 레벨에 두면 전체 메서드에 적용되는데, @RestController 에노테이션 안에 @ResponseBody 가 적용되어 있다.

<h3>HTTP 메시지 컨버터</h3>

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/ed58e7eb-7dce-4966-b590-54c8cc71bfeb)

ResponseBody 를 사용
1. HTTP의 BODY에 문자 내용을 직접 반환
2. viewResolver 대신에 HttpMessageConverter 가 동작
3. 기본 문자처리: StringHttpMessageConverter
4. 기본 객체처리: MappingJackson2HttpMessageConverter
5. byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

<h5>스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다</h5>
- HTTP 요청: @RequestBody , HttpEntity(RequestEntity) , 
- HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity) , 

```
package org.springframework.http.converter;
public interface HttpMessageConverter<T> {
      boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
      boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
      List<MediaType> getSupportedMediaTypes();
      T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
      throws IOException, HttpMessageNotReadableException;
      void write(T t, @Nullable MediaType contentType, HttpOutputMessage
      outputMessage)
      throws IOException, HttpMessageNotWritableException;
}
```

- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.
- canRead() , canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read() , write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

몇가지 주요한 메시지 컨버터를 알아보자.
1. ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다.
- 클래스 타입: byte[] , 미디어타입: */* ,
- 요청 예) @RequestBody byte[] data
- 응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream

2. StringHttpMessageConverter : String 문자로 데이터를 처리한다.
- 클래스 타입: String , 미디어타입: */*
- 요청 예) @RequestBody String data
- 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain

```
content-type: application/json
@RequestMapping
void hello(@RequestBody String data) {}
```

3.MappingJackson2HttpMessageConverter : application/json
- 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
- 요청 예) @RequestBody HelloData data
- 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

```
content-type: application/json
@RequestMapping
void hello(@RequestBody HelloData data) {
```


<h5>HTTP 요청 데이터 읽기</h5>
- HTTP 요청이 오고, 컨트롤러에서 @RequestBody , HttpEntity 파라미터를 사용한다.
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출한다.
    - 대상 클래스 타입을 지원하는가.
      - 예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )
    - HTTP 요청의 Content-Type 미디어 타입을 지원하는가.
      - 예) text/plain , application/json , */*
- canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다.

<h5>HTTP 응답 데이터 생성</h5>
- 컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다. 
- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.
    - 대상 클래스 타입을 지원하는가.
        - 예) return의 대상 클래스 ( byte[] , String , HelloData )
    - HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces )
        - 예) text/plain , application/json , */*
- canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다

<h3>요청 매핑 헨들러 어뎁터 구조</h3>

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/f2a4ab54-f389-417a-ac6b-d30c565acc01)

- 모든 비밀은 애노테이션 기반의 컨트롤러, 그러니까 @RequestMapping 을 처리하는 핸들러 어댑터인
RequestMappingHandlerAdapter (요청 매핑 헨들러 어뎁터)에 있다

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/5188264e-7def-43a6-ab52-9c4cd4255826)

<h5>ArgumentResolver</h5>
- 생각해보면, 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
- HttpServletRequest , Model 은 물론이고, @RequestParam , @ModelAttribute 같은 애노테이션 그리고 @RequestBody , HttpEntity 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보여주었다.
- 이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 ArgumentResolver 덕분이다.
- 애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter 는 바로 이 ArgumentResolver 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다. 그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다

```
public interface HandlerMethodArgumentResolver {
    boolean supportsParameter(MethodParameter parameter);
    @Nullable
    Object resolveArgument(MethodParameter parameter, @Nullable 
    ModelAndViewContainer mavContainer,
    NativeWebRequest webRequest, @Nullable WebDataBinderFactory
    binderFactory) throws Exception;
}
```

동작 방식
- ArgumentResolver 의 supportsParameter() 를 호출해서 해당 파라미터를 지원하는지 체크하고, 지원하면 resolveArgument() 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러 호출시 넘어가는 것이다.
- 그리고 원한다면 여러분이 직접 이 인터페이스를 확장해서 원하는 ArgumentResolver 를 만들 수도 있다.

<h5>ReturnValueHandler</h5>
- HandlerMethodReturnValueHandler 를 줄여서 ReturnValueHandler 라 부른다.
- ArgumentResolver 와 비슷한데, 이것은 응답 값을 변환하고 처리한다. 컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ReturnValueHandler 덕분이다.

<h3>HTTP 메시지 컨버터</h3>
![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/52e3891c-5540-4bc1-8d41-d280ad71a150)

- HTTP 메시지 컨버터를 사용하는 @RequestBody 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.
@ResponseBody 의 경우도 컨트롤러의 반환 값을 이용한다.
- 요청의 경우 @RequestBody 를 처리하는 ArgumentResolver 가 있고, HttpEntity 를 처리하는
ArgumentResolver 가 있다. 이 ArgumentResolver 들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.

- 응답의 경우 @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler 가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.
- 스프링 MVC는 @RequestBody @ResponseBody 가 있으면 RequestResponseBodyMethodProcessor()
HttpEntity 가 있으면 HttpEntityMethodProcessor() 를 사용한다

<h4>확장</h4>
- 스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 확장할 수 있다.
1. HandlerMethodArgumentResolver
2. HandlerMethodReturnValueHandler
3. HttpMessageConverter

-> 스프링이 필요한 대부분의 기능을 제공하기 때문에 실제 기능을 확장할 일이 많지는 않다. 기능 확장은 WebMvcConfigurer 를 상속 받아서 스프링 빈으로 등록하면 된다

```
@Bean
public WebMvcConfigurer webMvcConfigurer() {
     return new WebMvcConfigurer() {
     @Override
     public void addArgumentResolvers(List<HandlerMethodArgumentResolver> 
    resolvers) {
     //...
     }
     @Override
     public void extendMessageConverters(List<HttpMessageConverter<?>> 
    converters) {
     //...
     }
     };
}
```

<h1>스프링 MVC - 웹 페이지 만들</h1>

```
import hello.itemservice.domain.item.Item;
import hello.itemservice.domain.item.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import javax.annotation.PostConstruct;
import java.util.List;
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {
 private final ItemRepository itemRepository;
 @GetMapping
 public String items(Model model) {
 List<Item> items = itemRepository.findAll();
 model.addAttribute("items", items);
 return "basic/items";
 }
 /**
 * 테스트용 데이터 추가
 */
 @PostConstruct
 public void init() {
   itemRepository.save(new Item("testA", 10000, 10));
   itemRepository.save(new Item("testB", 20000, 20));
 }
}
```

@RequiredArgsConstructor
- final 이 붙은 멤버변수만 사용해서 생성자를 자동으로 만들어준다.

<h4>테스트용 데이터 추가</h4>
- 테스트용 데이터가 없으면 회원 목록 기능이 정상 동작하는지 확인하기 어렵다.
- @PostConstruct : 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다.


<h3>타임리프 간단히 알아보기</h3>
<h5>속성 변경 - th:href</h5>
- th:href="@{/css/bootstrap.min.css}"
1. href="value1" 을 th:href="value2" 의 값으로 변경한다.
2. 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 만약 값이 없다면 새로 생성한다.
3. HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 th:href 의 값이 href 로 대체되면서 동적으로 변경할 수 있다.
4. 대부분의 HTML 속성을 th:xxx 로 변경할 수 있다.

<h5>타임리프 핵심</h5>
- 핵심은 th:xxx 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다. th:xxx 이 없으면 기존 html의 xxx 속성이 그대로 사용된다.
- HTML을 파일로 직접 열었을 때, th:xxx 가 있어도 웹 브라우저는 th: 속성을 알지 못하므로 무시한다.
- 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.

<h5>URL 링크 표현식 - @{...},</h5>
- th:href="@{/css/bootstrap.min.css}"
- @{...} : 타임리프는 URL 링크를 사용하는 경우 @{...} 를 사용한다. 이것을 URL 링크 표현식이라 한다.
- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다

<h5>속성 변경 - th:onclick</h5>
- onclick="location.href='addForm.html'"
- th:onclick="|location.href='@{/basic/items/add}'|"
여기에는 다음에 설명하는 리터럴 대체 문법이 사용되었다. 자세히 알아보자.

<h5>리터럴 대체 - |...|</h5>
- |...| :이렇게 사용한다.
- 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">

  다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
<span th:text="|Welcome to our application, ${user.name}!|">

결과를 다음과 같이 만들어야 하는데
location.href='/basic/items/add'
그냥 사용하면 문자와 표현식을 각각 따로 더해서 사용해야 하므로 다음과 같이 복잡해진다.
th:onclick="'location.href=' + '\'' + @{/basic/items/add} + '\''"

리터럴 대체 문법을 사용하면 다음과 같이 편리하게 사용할 수 있다.
th:onclick="|location.href='@{/basic/items/add}'|"

<h5>반복 출력 - th:each</h5>
<tr th:each="item : ${items}">
반복은 th:each 를 사용한다. 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 하나씩 포함되고, 반복문 안에서 item 변수를 사용할 수 있다.
컬렉션의 수 만큼 <tr>..</tr> 이 하위 테그를 포함해서 생성된다.

<h5>변수 표현식 - ${...}</h5>
<td th:text="${item.price}">10000</td>
모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
프로퍼티 접근법을 사용한다. ( item.getPrice() )

<h5>내용 변경 - th:text</h5>
<td th:text="${item.price}">10000</td>
내용의 값을 th:text 의 값으로 변경한다.
여기서는 10000을 ${item.price} 의 값으로 변경한다.
<h5>URL 링크 표현식2 - @{...}, </h5>
- th:href="@{/basic/items/{itemId}(itemId=${item.id})}"
- 상품 ID를 선택하는 링크를 확인해보자.
- URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.
- 경로 변수( {itemId} ) 뿐만 아니라 쿼리 파라미터도 생성한다.
- 예) th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"
- 생성 링크: http://localhost:8080/basic/items/1?query=test
<h5>URL 링크 간단히</h5>
- th:href="@{|/basic/items/${item.id}|}"
- 상품 이름을 선택하는 링크를 확인해보자.
- 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.

-> 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿(natural templates)이라 한다

<h4>상품 등록 처리 - @ModelAttribute</h4>

```
/**
 * @ModelAttribute("item") Item item
 * model.addAttribute("item", item); 자동 추가
 */
@PostMapping("/add")
public String addItemV2(@ModelAttribute("item") Item item, Model model) {
   itemRepository.save(item);
   //model.addAttribute("item", item); //자동 추가, 생략 가능
   return "basic/item";
}
```

1. @ModelAttribute - 요청 파라미터 처리
2. @ModelAttribute 는 Item 객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다.
3. @ModelAttribute - Model 추가
4. @ModelAttribute 는 중요한 한가지 기능이 더 있는데, 바로 모델(Model)에 @ModelAttribute 로 지정한 객체를 자동으로 넣어준다. 지금 코드를 보면 model.addAttribute("item", item) 가 주석처리 되어 있어도 잘 동작하는 것을 확인할 수 있다.
5. 모델에 데이터를 담을 때는 이름이 필요하다. 이름은 @ModelAttribute 에 지정한 name(value) 속성을 사용한다. 만약 다음과 같이 @ModelAttribute 의 이름을 다르게 지정하면 다른 이름으로 모델에 포함된다.
6. @ModelAttribute("hello") Item item 이름을 hello 로 지정
- model.addAttribute("hello", item); 모델에 hello 이름으로 저장

<h3>PRG Post/Redirect/Get</h3>
- 사실 지금까지 진행한 상품 등록 처리 컨트롤러는 심각한 문제가 있다. (addItemV1 ~ addItemV4)
상품 등록을 완료하고 웹 브라우저의 새로고침 버튼을 클릭해보자.
- 상품이 계속해서 중복 등록되는 것을 확인할 수 있다.

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/1390ba7d-d5d8-4f3e-a6f5-acfd7e7dbfc8)

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-
Technology_1/assets/82505269/10e69a0f-c6b6-42b7-90ee-aa66e470e82c)

![image](https://github.com/kimSM94/Spring-MVC-Part-1_Backend-Web-Development-Key-Technology_1/assets/82505269/2bf9ba08-ebb7-4547-898c-2284c27e3056)


```
/**
 * PRG - Post/Redirect/Get
 */
@PostMapping("/add")
public String addItemV5(Item item) {
 itemRepository.save(item);
 return "redirect:/basic/items/" + item.getId();
}
```

- 상품 상세 화면으로 리다이렉트를 호출해주면 된다
  -> 이런 문제 해결 방식을 PRG Post/Redirect/Get

※ 주의
redirect:/basic/items/" + item.getId() redirect에서 +item.getId() 처럼 URL에 변수를 더해서 사용하는 것은 URL 인코딩이 안되기 때문에 위험하다. 다음에 설명하는 RedirectAttributes 를 사용하자

<h4>RedirectAttributes</h4>
상품을 저장하고 상품 상세 화면으로 리다이렉트 한 것 까지는 좋았다. 그런데 고객 입장에서 저장이 잘 된 것인지 안 된 것인지 확신이 들지 않는다. 그래서 저장이 잘 되었으면 상품 상세 화면에 "저장되었습니다"라는 메시지를 보여달라는 요구사항이 왔다. 간단하게 해결해보자

```
/**
 * RedirectAttributes
 */
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
 Item savedItem = itemRepository.save(item);
 redirectAttributes.addAttribute("itemId", savedItem.getId());
 redirectAttributes.addAttribute("status", true);
 return "redirect:/basic/items/{itemId}";
}
```

```
<div class="container">
 <div class="py-5 text-center">
 <h2>상품 상세</h2>
 </div>
 <!-- 추가 -->
 <h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
```

조건에 맞으면 출력
