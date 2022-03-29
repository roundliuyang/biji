```
package com.yly.resttemplate;



import java.net.URI;
import java.util.Arrays;
import java.util.Set;




/**
 * Author: yly
 * Date: 2021/11/6 16:14
 *
 * 从 Spring Framework 5 开始，除了 WebFlux 堆栈之外，Spring 还引入了一个名为WebClient的新 HTTP 客户端。
 * WebClient 是RestTemplate的现代替代 HTTP 客户端。它不仅提供了传统的同步 API，而且还支持高效的非阻塞和异步方法。
 * 也就是说，如果我们正在开发新应用程序或迁移旧应用程序，最好使用 WebClient。展望未来，RestTemplate 将在未来版本中弃用。
 */
public class resttemplateTest {

    private RestTemplate restTemplate;


    private static final String fooResourceUrl = "http://localhost:" + APPLICATION_PORT + "/spring-rest/foos";
    private static final String FOO_RESOURCE_URL = "http://localhost:" + 8082 + "/spring-rest/foos";
    private static final String URL_SECURED_BY_AUTHENTICATION = "http://httpbin.org/basic-auth/user/passwd";
    private static final String BASE_URL = "http://localhost:" + 8082 + "/spring-rest";

    //使用Get检索资源

    // 获取纯 JSON
    @Test
    public  void test() throws InterruptedException {
        RestTemplate restTemplate = new RestTemplate();
        String fooResourceUrl
                = "http://localhost:8080/spring-rest/foos";
        ResponseEntity<String> response
                = restTemplate.getForEntity(fooResourceUrl + "/1", String.class);
        System.out.println(response);
        Thread.sleep(80L);

    }

    /*
        请注意，我们拥有对 HTTP 响应的完全访问权限，因此我们可以执行诸如检查状态代码以确保操作成功或使用响应的实际正文之类的操作：
        ObjectMapper mapper = new ObjectMapper();
        JsonNode root = mapper.readTree(response.getBody());
        JsonNode name = root.path("name");
        assertThat(name.asText(), notNullValue());
        我们在这里将响应正文作为标准字符串使用，并使用 Jackson（以及 Jackson 提供的 JSON 节点结构）来验证一些细节。
     */

    // Retrieving POJO Instead of JSON
    // 我们还可以将响应直接映射到资源 DTO：
    @Test
    public void givenTestRestTemplate_whenSendGetForEntity_thenStatusOk() {
        TestRestTemplate testRestTemplate = new TestRestTemplate();
        ResponseEntity<Foo> response = testRestTemplate.getForEntity(FOO_RESOURCE_URL + "/1", Foo.class);
        assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));
    }

    // HEAD
   @Test
    public void givenFooService_whenCallHeadForHeaders_thenReceiveAllHeaders() {
        TestRestTemplate testRestTemplate = new TestRestTemplate();
        final HttpHeaders httpHeaders = testRestTemplate.headForHeaders(FOO_RESOURCE_URL);
        assertTrue(httpHeaders.getContentType().includes(MediaType.APPLICATION_JSON));
    }



   /*
        使用POST创建资源
        为了在 API 中创建新的 Resource，我们可以充分利用postForLocation()、postForObject()或postForEntity() API。
        第一个返回新创建的资源的 URI，而第二个返回资源本身。
    */

    // The postForObject() API
    @Test
    public void givenFooService_whenPostForObject_thenCreatedObjectIsReturned() {
        final HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"));
        final Foo foo = restTemplate.postForObject(fooResourceUrl, request, Foo.class);
        assertThat(foo, notNullValue());
        assertThat(foo.getName(), is("bar"));
    }


    // The postForLocation() API
    // 它没有返回完整的资源，而只是返回新创建的资源的位置。
    @Test
    public void givenFooService_whenPostForLocation_thenCreatedLocationIsReturned() {
        final HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"));
        final URI location = restTemplate.postForLocation(fooResourceUrl, request, Foo.class);
        assertThat(location, notNullValue());
    }


    // The exchange() API
    // Let's have a look at how to do a POST with the more generic exchange API:

    @Test
    public void test2(){
        RestTemplate restTemplate = new RestTemplate();
        HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"));
        ResponseEntity<Foo> response = restTemplate
                .exchange(fooResourceUrl, HttpMethod.POST, request, Foo.class);

        assertThat(response.getStatusCode(), is(HttpStatus.CREATED));

        Foo foo = response.getBody();

        assertThat(foo, notNullValue());
        assertThat(foo.getName(), is("bar"));
    }


    //  Submit Form Data
    // 接下来，让我们看看如何使用 POST 方法提交表单。

    @Test
    public void givenFooService_whenFormSubmit_thenResourceIsCreated() {

        //首先，我们需要将Content-Type标头设置为application/x-www-form-urlencoded
        //这确保可以将大查询字符串发送到服务器，其中包含由&分隔的 名称/值对：
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

        // 我们可以将表单变量包装到LinkedMultiValueMap 中：
        MultiValueMap<String, String> map= new LinkedMultiValueMap<>();
        map.add("id", "10");

        // 接下来，我们使用HttpEntity实例构建请求 ：
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(map, headers);

        // Finally, we can connect to the REST service by calling restTemplate.postForEntity() on the Endpoint: /foos/form
        ResponseEntity<String> response = restTemplate.postForEntity( fooResourceUrl+"/form", request , String.class);

        assertThat(response.getStatusCode(), is(HttpStatus.CREATED));
        final String fooResponse = response.getBody();
        assertThat(fooResponse, notNullValue());
        assertThat(fooResponse, is("10"));
    }


    //  使用 OPTIONS 获取允许的操作
    // Next, we're going to have a quick look at using an OPTIONS request and exploring the allowed operations on a specific URI using this kind of request;
    // the API is optionsForAllow:

    @Test
    public void givenFooService_whenCallOptionsForAllow_thenReceiveValueOfAllowHeader() {
        final Set<HttpMethod> optionsForAllow = restTemplate.optionsForAllow(fooResourceUrl);
        final HttpMethod[] supportedMethods = { HttpMethod.GET, HttpMethod.POST, HttpMethod.HEAD };

        assertTrue(optionsForAllow.containsAll(Arrays.asList(supportedMethods)));
    }

    //  7.使用 PUT 更新资源
    // 接下来，我们将开始研究PUT，更具体地说，是这个操作的exchange()API，因为template.put API是非常简单明了的

    @Test
    public void givenFooService_whenPutExistingEntity_thenItIsUpdated() {
        final HttpHeaders headers=null;             //= prepareBasicAuthHeaders();
        final HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"), headers);

        // Create Resource
        final ResponseEntity<Foo> createResponse = restTemplate.exchange(fooResourceUrl, HttpMethod.POST, request, Foo.class);

        // Update Resource
        final Foo updatedInstance = new Foo("newName");
        updatedInstance.setId(createResponse.getBody()
                .getId());
        final String resourceUrl = fooResourceUrl + '/' + createResponse.getBody()
                .getId();
        final HttpEntity<Foo> requestUpdate = new HttpEntity<>(updatedInstance, headers);
        restTemplate.exchange(resourceUrl, HttpMethod.PUT, requestUpdate, Void.class);

        // Check that Resource was updated
        final ResponseEntity<Foo> updateResponse = restTemplate.exchange(resourceUrl, HttpMethod.GET, new HttpEntity<>(headers), Foo.class);
        final Foo foo = updateResponse.getBody();
        assertThat(foo.getName(), is(updatedInstance.getName()));
    }


    // 8. 使用 DELETE 删除资源
    // 要删除现有资源，我们将快速使用delete() API：
//    String entityUrl = fooResourceUrl + "/" + existingResource.getId();
//    restTemplate.delete(entityUrl);




//         9. 配置超时
//         我们可以通过简单地使用ClientHttpRequestFactory来配置RestTemplate超时：

//    RestTemplate restTemplate = new RestTemplate(getClientHttpRequestFactory());
    private ClientHttpRequestFactory getClientHttpRequestFactory() {
        int timeout = 5000;
        HttpComponentsClientHttpRequestFactory clientHttpRequestFactory
                = new HttpComponentsClientHttpRequestFactory();
        clientHttpRequestFactory.setConnectTimeout(timeout);
        return clientHttpRequestFactory;
    }
    // 我们可以使用HttpClient进行进一步的配置选项：

/*
    private ClientHttpRequestFactory getClientHttpRequestFactory() {
        int timeout = 5000;
        RequestConfig config = RequestConfig.custom()
                .setConnectTimeout(timeout)
                .setConnectionRequestTimeout(timeout)
                .setSocketTimeout(timeout)
                .build();
        CloseableHttpClient client = HttpClientBuilder
                .create()
                .setDefaultRequestConfig(config)
                .build();
        return new HttpComponentsClientHttpRequestFactory(client);
    }
 */


    // 10。  ParameterizedType
    /**
         假设接收请求的类如下。
         public class ResultClass {
         private long idx;
         private String name;
         }
         如何以列表形式获得回报 - 存在问题。不建议
         List<Resultclass> list = restTemplate.getForObject("url", List.class);
         这种方法是有问题的。
         编译没有问题，但是调用后查看list 值，可以看到它有一个LinkedHashMap对象，而不是ResultClass对象。
         所以，如果你像下面这样调用它，你调用它的那一刻就会发生错误。
         List<Resultclass> list = restTemplate.getForObject("url", List.class);
         ResultClass resultClass = list.get(0);
         resultClass.getIdx();
         如果返回为List<map<Object, Object>>并被使用，可以这样调用，但不建议再次转换，不方便。

         在交换中使用 ParameterizedTypeReference
         springframework 3.2之后，支持ParameterizedTypeReference，可以如下使用。
         ResponseEntity<List<ResultClass>> response = restTemplate.exchange("url",HttpMethod.GET, null, new ParameterizedTypeReference<List<ResultClass>>() {});
         List<ResultClass> list = response.getBody();
         同样的，非数组情况同样适用
     */


     // 11. 以数组形式返回
     /*
         如果使用 xxxForObject 方法，如下所示。
         ResultClass[] resultClasses = restTemplate.getForObject("url", ResultClass[].class); List<Resultclass> list = Arrays.asList(resultClasses);

         如果使用 xxxForEntity，则如下。
         ResponseEntity<ResultClass[]> responseEntity = restTemplate.getForEntity("url", ResultClass[].class);
         List<Resultclass> list = Arrays.asList(responseEntity.getBody());
     */

}

```