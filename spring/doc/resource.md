# 탄생 배경

- classpath resource 접근 불편함
- ServletContext 기준 상대경로 접근 불편함

# 구현체

- UrlResource
- ClassPathResource(default: ClassPathXmlApplicationContext)
- FileSystemResource(default: FileSystemXmlApplicationContext)
- ServletContextResource(default: WebApplicationContext)

# 사용법

- ApplicationContext 종류에 따라 Url접두어 없이 쓰거나 붙혀서 써야 함

```java

// WebApplicationContext 인 경우
@Component
public class ResourceAppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("== resources ==");

        System.out.println(resourceLoader.getClass()); // AnnotationConfigServletWebServerApplicationContext

        Resource servletContextResource = resourceLoader.getResource("test.txt");
        System.out.println(servletContextResource.getClass()); // ServletContextResource

        Resource classPathResource = resourceLoader.getResource("classpath:test.txt");
        System.out.println(classPathResource.getClass()); // ClassPathResource

        System.out.println("== resources ==");
    }
}
```
