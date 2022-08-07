We are going to establish cross namepsace communication across pods using calico and network policy in this tutorial.

Note: Assuming you have EKS cluster already running.


Steps:

1. Install postgress and springboot app
2. Install calico using helm
3. Update network policy to default deny  
4. Update network policy to allow communication from springboot to postgress



1. Install postgress and springboot app
2.

    # kubectl apply -f postgress/ -n backend
      
      namespace/backend created
      configmap/postgres-db-config created
      statefulset.apps/postgresql-db created
      service/postgres-db created


    # kubectl create configmap hostname-config --from-literal=postgres_host=$(kubectl get svc postgres-db -n backend -o jsonpath="{.spec.clusterIP}") -n frontend


    # k apply -f springboot/ -n frontend

    namespace/frontend unchanged
    secret/postgres-secrets unchanged
    deployment.apps/spring-boot-postgres-sample created
    service/spring-boot-postgres-sample unchanged

    # kubectl get all -n frontend

    NAME                                              READY   STATUS    RESTARTS   AGE
    pod/spring-boot-postgres-sample-f8975578d-fxskr   1/1     Running   0          6m4s

    NAME                                  TYPE           CLUSTER-IP     EXTERNAL-IP                                                              PORT(S)          AGE
    service/spring-boot-postgres-sample   LoadBalancer   10.100.69.60   a072263482e9f4616a27960a632a637f-545447142.us-east-2.elb.amazonaws.com   8080:31389/TCP   7m5s

    NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/spring-boot-postgres-sample   1/1     1            1           6m4s

    NAME                                                    DESIRED   CURRENT   READY   AGE
    replicaset.apps/spring-boot-postgres-sample-f8975578d   1         1         1       6m4s


    # kubectl get all -n backend

    NAME                  READY   STATUS    RESTARTS   AGE
    pod/postgresql-db-0   1/1     Running   0          6m32s
    pod/postgresql-db-1   1/1     Running   0          4m11s

    NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    service/postgres-db   ClusterIP   10.100.43.111   <none>        5432/TCP   6m32s

    NAME                             READY   AGE
    statefulset.apps/postgresql-db   2/2     6m32s
  
  
Till now application is working properly.
  
    # k logs   spring-boot-postgres-sample-f8975578d-fxskr -n frontend -f

      .   ____          _            __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
     \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |_\__, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::        (v2.2.1.RELEASE)

    2022-08-07 06:04:52.046  INFO 1 --- [           main] c.e.p.PostgresDemoApplication            : Starting PostgresDemoApplication v0.0.1-SNAPSHOT on spring-boot-postgres-sample-f8975578d-fxskr with PID 1 (/app.jar started by root in /)
    2022-08-07 06:04:52.050  INFO 1 --- [           main] c.e.p.PostgresDemoApplication            : No active profile set, falling back to default profiles: default
    2022-08-07 06:04:53.002  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data repositories in DEFAULT mode.
    2022-08-07 06:04:53.077  INFO 1 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 64ms. Found 2 repository interfaces.
    2022-08-07 06:04:53.658  INFO 1 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
    2022-08-07 06:04:54.082  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
    2022-08-07 06:04:54.106  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
    2022-08-07 06:04:54.106  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.27]
    2022-08-07 06:04:54.198  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
    2022-08-07 06:04:54.198  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2048 ms
    2022-08-07 06:04:54.769  INFO 1 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
    2022-08-07 06:04:54.860  INFO 1 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.4.8.Final}
    2022-08-07 06:04:55.054  INFO 1 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.0.Final}
    2022-08-07 06:04:55.233  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
    2022-08-07 06:04:55.485  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
    2022-08-07 06:04:55.519  INFO 1 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.PostgreSQL10Dialect
    2022-08-07 06:04:57.137  INFO 1 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
    2022-08-07 06:04:57.146  INFO 1 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
    2022-08-07 06:04:57.784  WARN 1 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
    2022-08-07 06:04:57.989  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
    2022-08-07 06:04:58.298  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
    2022-08-07 06:04:58.301  INFO 1 --- [           main] c.e.p.PostgresDemoApplication            : Started PostgresDemoApplication in 7.077 seconds (JVM running for 7.653)
    2022-08-07 06:07:00.834  INFO 1 --- [nio-8080-exec-6] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
    2022-08-07 06:07:00.834  INFO 1 --- [nio-8080-exec-6] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
    2022-08-07 06:07:00.842  INFO 1 --- [nio-8080-exec-6] o.s.web.servlet.DispatcherServlet        : Completed initialization in 8 ms
  

2.Install calico using helm


    # kubectl create namespace tigera-operator

    namespace/tigera-operator created
    [root@ip-172-31-3-61 opt]# helm install calico projectcalico/tigera-operator --version v3.23.3 --namespace tigera-operator
    W0807 03:47:15.421088    3852 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
    W0807 03:47:15.679160    3852 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
    NAME: calico
    LAST DEPLOYED: Sun Aug  7 03:47:13 2022
    NAMESPACE: tigera-operator
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None


3. Update network policy to default deny  
4. Update network policy to allow communication from springboot to postgress
