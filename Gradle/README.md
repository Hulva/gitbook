# Gradle

## `gradle.properties` Gradle proxy
```
systemProp.http.proxyHost=10.16.46.161
systemProp.http.proxyPort=3333
systemProp.https.proxyHost=10.16.46.161
systemProp.https.proxyPort=3333
```

## repositories
```
repositories {
    mavenLocal()
    // For license plugin.
    maven {
      url 'http://dl.bintray.com/content/netflixoss/external-gradle-plugins/'
    }
    maven {
      url 'http://10.16.46.161:8888/nexus/content/groups/ec_maven/'
    }
    maven {
      url 'http://10.16.46.161:8888/nexus/content/repositories/central/'
    }
    maven {
      url 'http://10.16.46.161:8888/nexus/content/repositories/ec_snapshots/'
    }
    maven {
      url 'http://10.16.46.161:8888/nexus/content/repositories/thirdparty/'
    }
    maven {
      url 'http://10.16.46.161:8888/nexus/content/repositories/releases/'
    }
}
```

