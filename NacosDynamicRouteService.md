```java
@Component
@Slf4j
public class NacosDynamicRouteService implements ApplicationEventPublisherAware {
  private String dataId = "gateway-router";
  private String group = "DEFAULT_GROUP";
  @Value("${spring.cloud.nacos.config.server-addr}")
  private String serverAddr;
  @Autowired
  private RouteDefinitionWriter routeDefinitionWriter;
  private ApplicationEventPublisher applicationEventPublisher;
  private static final List<String> ROUTE_LIST = new ArrayList<>();
  @PostConstruct
  public void dynamicRouteByNacosListener() {
    try {
      ConfigService configService = NacosFactory.createConfigService(serverAddr);
      configService.getConfig(dataId, group, 5000);
      configService.addListener(dataId, group, new Listener() {
        @Override
        public void receiveConfigInfo(String configInfo) {
          clearRoute();
          try {
            if (StringUtil.isNullOrEmpty(configInfo)) {//配置被删除
              return;
            }
            List<RouteDefinition> gatewayRouteDefinitions = JSONObject.parseArray(configInfo, RouteDefinition.class);
            for (RouteDefinition routeDefinition : gatewayRouteDefinitions) {
              addRoute(routeDefinition);
            }
            publish();
          } catch (Exception e) {
            log.error("receiveConfigInfo error" + e);
          }
        }
        @Override
        public Executor getExecutor() {
          return null;
        }
      });
    } catch (NacosException e) {
        log.error("dynamicRouteByNacosListener error" + e);
    }
  }
  private void clearRoute() {
    for (String id : ROUTE_LIST) {
      this.routeDefinitionWriter.delete(Mono.just(id)).subscribe();
    }
    ROUTE_LIST.clear();
  }
  private void addRoute(RouteDefinition definition) {
    try {
      routeDefinitionWriter.save(Mono.just(definition)).subscribe();
      ROUTE_LIST.add(definition.getId());
    } catch (Exception e) {
 log.error("addRoute error" + e);
    }
  }
  private void publish() {
    this.applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this.routeDefinitionWriter));
  }
  @Override
  public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
    this.applicationEventPublisher = applicationEventPublisher;
  }
```
