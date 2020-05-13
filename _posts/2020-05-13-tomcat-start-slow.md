---
layout: post
title: tomcat启动非常慢原因深入分析
categories: Java
description: tomcat启动非常慢原因深入分析
keywords: tomcat
---

有些情况下tomcat启动非常慢，通过jstack查看当前堆栈


/opt/java/jdk1.8.0_121/bin/jstack  14970 > /home/ubuntu/j.log


关键内容

```
"main" #1 prio=5 os_prio=0 tid=0x00007fc69c00a000 nid=0x3a7b runnable [0x00007fc6a5db5000]
java.lang.Thread.State: RUNNABLE
at java.io.FileInputStream.readBytes(Native Method)
at java.io.FileInputStream.read(FileInputStream.java:255)
at sun.security.provider.SeedGenerator$URLSeedGenerator.getSeedBytes(SeedGenerator.java:539)
at sun.security.provider.SeedGenerator.generateSeed(SeedGenerator.java:144)
at sun.security.provider.SecureRandom$SeederHolder.<clinit>(SecureRandom.java:203)
at sun.security.provider.SecureRandom.engineNextBytes(SecureRandom.java:221)
- locked <0x00000006883eb138> (a sun.security.provider.SecureRandom)
at java.security.SecureRandom.nextBytes(SecureRandom.java:468)
at java.security.SecureRandom.next(SecureRandom.java:491)
at java.util.Random.nextInt(Random.java:329)
at org.apache.catalina.util.SessionIdGeneratorBase.createSecureRandom(SessionIdGeneratorBase.java:266)
at org.apache.catalina.util.SessionIdGeneratorBase.getRandomBytes(SessionIdGeneratorBase.java:203)
at org.apache.catalina.util.StandardSessionIdGenerator.generateSessionId(StandardSessionIdGenerator.java:34)
at org.apache.catalina.util.SessionIdGeneratorBase.generateSessionId(SessionIdGeneratorBase.java:195)
at org.apache.catalina.util.SessionIdGeneratorBase.startInternal(SessionIdGeneratorBase.java:285)
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
- locked <0x00000006883ead88> (a org.apache.catalina.util.StandardSessionIdGenerator)
```

分下SeedGenerator源码，URLSeedGenerator为SeedGenerator的内部类

```java
static class URLSeedGenerator extends SeedGenerator {
        private String deviceName;
        private InputStream seedStream;

        URLSeedGenerator(String var1) throws IOException {
            if(var1 == null) {
                throw new IOException("No random source specified");
            } else {
                this.deviceName = var1;
                this.init();
            }
        }

        private void init() throws IOException {
            final URL var1 = new URL(this.deviceName);

            try {
                this.seedStream = (InputStream)AccessController.doPrivileged(new PrivilegedExceptionAction() {
                    public InputStream run() throws IOException {
                        if(var1.getProtocol().equalsIgnoreCase("file")) {
                            File var1x = SunEntries.getDeviceFile(var1);
                            return new FileInputStream(var1x);
                        } else {
                            return var1.openStream();
                        }
                    }
                });
            } catch (Exception var3) {
                throw new IOException("Failed to open " + this.deviceName, var3.getCause());
            }
        }

        void getSeedBytes(byte[] var1) {
            int var2 = var1.length;
            int var3 = 0;

            try {
                while(var3 < var2) {
                    int var4 = this.seedStream.read(var1, var3, var2 - var3);
                    if(var4 < 0) {
                        throw new InternalError("URLSeedGenerator " + this.deviceName + " reached end of file");
                    }

                    var3 += var4;
                }

            } catch (IOException var5) {
                throw new InternalError("URLSeedGenerator " + this.deviceName + " generated exception: " + var5.getMessage(), var5);
            }
        }
    }
```

getSeedBytes函数使用seedStream读取数据，seedStream在init()函数中初始化，实际上代表的是deviceName文件输入流，具体deviceName的路径初始化由URLSeedGenerator的构造函数完成。

SeedGenerator的静态构造器中初始化了SeedGenerator.URLSeedGenerator

```java
static {
        String var0 = SunEntries.getSeedSource();
        if(!var0.equals("file:/dev/random") && !var0.equals("file:/dev/urandom")) {
            if(var0.length() != 0) {
                try {
                    instance = new SeedGenerator.URLSeedGenerator(var0);
                    if(debug != null) {
                        debug.println("Using URL seed generator reading from " + var0);
                    }
                } catch (IOException var2) {
                    if(debug != null) {
                        debug.println("Failed to create seed generator with " + var0 + ": " + var2.toString());
                    }
                }
            }
        } else {
            try {
                instance = new NativeSeedGenerator(var0);
                if(debug != null) {
                    debug.println("Using operating system seed generator" + var0);
                }
            } catch (IOException var3) {
                if(debug != null) {
                    debug.println("Failed to use operating system seed generator: " + var3.toString());
                }
            }
        }

        if(instance == null) {
            if(debug != null) {
                debug.println("Using default threaded seed generator");
            }

            instance = new SeedGenerator.ThreadedSeedGenerator();
        }

    }
```

如果 SunEntries.getSeedSource() 返回内容不为空，那么文件就是"file:/dev/random" 或者 "file:/dev/urandom"，否则是NativeSeedGenerator，从jstack的打印来看流程是走到 SeedGenerator.URLSeedGenerator中。

```java
final class SunEntries {
    private static final String PROP_EGD = "java.security.egd";
    private static final String PROP_RNDSOURCE = "securerandom.source";
    static final String URL_DEV_RANDOM = "file:/dev/random";
    static final String URL_DEV_URANDOM = "file:/dev/urandom";
    private static final String seedSource = (String)AccessController.doPrivileged(new PrivilegedAction() {
        public String run() {
            String var1 = System.getProperty("java.security.egd", "");
            if(var1.length() != 0) {
                return var1;
            } else {
                var1 = Security.getProperty("securerandom.source");
                return var1 == null?"":var1;
            }
        }
    });

    private SunEntries() {
    }

    static void putEntries(Map<Object, Object> var0) {
        boolean var1 = NativePRNG.isAvailable();
        boolean var2 = seedSource.equals("file:/dev/urandom") || seedSource.equals("file:/dev/random");
        if(var1 && var2) {
            var0.put("SecureRandom.NativePRNG", "sun.security.provider.NativePRNG");
        }

        var0.put("SecureRandom.SHA1PRNG", "sun.security.provider.SecureRandom");
        if(var1 && !var2) {
            var0.put("SecureRandom.NativePRNG", "sun.security.provider.NativePRNG");
        }

        if(Blocking.isAvailable()) {
            var0.put("SecureRandom.NativePRNGBlocking", "sun.security.provider.NativePRNG$Blocking");
        }

        if(NonBlocking.isAvailable()) {
            var0.put("SecureRandom.NativePRNGNonBlocking", "sun.security.provider.NativePRNG$NonBlocking");
        }

        var0.put("Signature.SHA1withDSA", "sun.security.provider.DSA$SHA1withDSA");
        ....
    }

    static String getSeedSource() {
        return seedSource;
    }

    static File getDeviceFile(URL var0) throws IOException {
        try {
            URI var1 = var0.toURI();
            if(var1.isOpaque()) {
                URI var2 = (new File(System.getProperty("user.dir"))).toURI();
                String var3 = var2.toString() + var1.toString().substring(5);
                return new File(URI.create(var3));
            } else {
                return new File(var1);
            }
        } catch (URISyntaxException var4) {
            return new File(var0.getPath());
        }
    }
}
```

seedSource的初始化参考PrivilegedAction的run函数，如果系统设置java.security.egd属性，那么从此处取seedSource表示的文件，否则调用Security.getProperty("securerandom.source")。

Security.getProperty读取jre下面的配置文件/opt/java/jdk1.8.0_121/jre/lib/security/java.security，java.security配置文件配置了/dev/random，参考下图。

![](/images/posts/tomcat-start-slow/1.png)

 可能由于系统interrupt不足，导致在jdk在使用/dev/random时卡死。

注：想让System.getProperty("java.security.egd", "")不空，可以在Java启动参数中设置-Djava.security.egd=xxxx的方式

**解决方法：**

> 1、即在java程序启动参数中添加：-Djava.security.egd=file:/dev/./urandom，使用/dev/urandom生成随机数。

    注：Java5以后file:/dev/./urandom，注意两个/之间的.

> 2、或者直接修改jre/lib/security/java.security配置文件，指向file:/dev/urandom

/dev/random和/dev/urandom的区别可以参考 https://lwn.net/Articles/184925/

```
The /dev/randomdevice was specifically designed to block when the entropy pool had insufficient entropy to satisfy the request. The /dev/urandom device is provided as an alternative that generates very good random numbers and does not block (and is therefore not vulnerable to a denial of service). For any but the most sensitive applications (key generation being an obvious choice), /dev/urandom is the recommended source for random numbers.
```