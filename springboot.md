## 1、配置文件注入

在application-prod.yml（配置文件）中，假设有文件层级earthquake-mqtt，代码如下：

```yaml
earthquake:
  mqtt:
    client-id: 89gyoKE0b80779okox8i38146D4N4Z1R
    serverURIs:
      - tcp://124.114.152.250:1885
    user-name: t0_YRww_
    passwd: 8I4Sx+nER8
```

则在EarthquakeMqttProperties.java文件中:

```java
package com.ruoyi.framework.config.properties;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * 地震预警MQTT 服务配置
 * @author ydzhanghaitao
 */
@Data
@Component
@ConfigurationProperties(prefix = "earthquake.mqtt")
public class EarthquakeMqttProperties {

    private String clientId;

    private String userName;

    private String passwd;

    private String[] serverURIs;

}
```

使用 `@ConfigurationProperties(prefix = "earthquake.mqtt")` 可以将配置注入。



