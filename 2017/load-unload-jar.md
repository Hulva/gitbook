```
File file = new File(jarPath);
URLClassLoader classLoader = new URLClassLoader(new URL[] {file.toURI().toURL()}, getClass().getClassLoader());

// this is required to load file (such as spring/context.xml) into the jar
// Thread.currentThread().setContextClassLoader(classLoader);
// ClassLoader classLoader;
// classLoader = URLClassLoader.newInstance(new URL[] {file.toURI().toURL()}, getClass().getClassLoader());

Class<?> clazz = Class.forName(classFullPath, true, classLoader);
Class<? extends AbstractLogMessageHandler> runClass = clazz.asSubclass(AbstractLogMessageHandler.class);
// Avoid Class.newInstance, for it is evil.
return runClass.getConstructor(JSONObject.class, String.class);
```
Close and Restart the ClassReloader
```
classLoader.close();
```