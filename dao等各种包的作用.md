- dao:	与数据库的操作
- model（entity）:     一般都是Javabean对象，例如与数据库某个表相关联
- service：供外部调用，对dao、model进行封装。一般在service的实现类会出现在action中，service提供了包含逻辑的数据库访问
- impl：service的实现类，或数据库访问等接口的实现类，再service中会通过spring注入这些接口来实现逻辑
- util：通常是工具类，如字符串处理、日期处理等

协作关系：

web页面——》action——》service——》interface——》impl——》dao——》数据库