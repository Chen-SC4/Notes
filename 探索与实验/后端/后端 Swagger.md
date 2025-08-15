## 1. 常用注解


- `@ApiOperation` Swagger 或 Knife4j 注解，用于在 API 文档中为 API 提供一个简短的标题
	- `@ApiOperationSupport` 对 `@ApiOperation` 中信息的补充
- `@ApiImplicitParams` Swagger 或 Knife4j 注解，是 `@ApiImplicitParam` 注解的容器
	- `@ApiImplicitParams` 描述方法中的一个参数，该注解用于手动描述参数，防止 Swagger 无法自动识别参数
- `@Api` Swagger 或 Knife4j 注解，用于描述一个 Controller 资源类，其核心属性为 `tags`，它的值会成为 API 文档左侧菜单栏中的一个分组标题。
	- `@ApiSupport` 是对 `@Api` 中信息的补充，其核心属性为 order 和 author。order 用于指定方法在 API 文档中的排序，author 指定 API 文档中对应方法的作者。
