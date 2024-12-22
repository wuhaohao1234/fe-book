## 设计理念
+ Formily：Schema 驱动，基于 JSON Schema 来驱动表单的生成和渲染，强调领域模型和 UI 逻辑的分离，适合构建复杂业务场景的表单
    - **<font style="color:rgb(51, 51, 51);">动态表单</font>**<font style="color:rgb(51, 51, 51);">：支持动态添加和移除字段、联动逻辑处理非常强大</font>
    - **<font style="color:rgb(51, 51, 51);">复杂验证</font>**<font style="color:rgb(51, 51, 51);">：能够处理复杂的验证场景，支持异步验证</font>
    - **<font style="color:rgb(51, 51, 51);">状态管理</font>**<font style="color:rgb(51, 51, 51);">：内置了复杂的状态管理机制，能够自动处理表单的状态和生命周期</font>
+ Ant Design Form： 组件驱动，更接近传统的组件化思路，开发者需要以组件的方式去创建表单
    - **<font style="color:rgb(51, 51, 51);">良好与 UI 组件的结合</font>**<font style="color:rgb(51, 51, 51);">：可以轻松与其他 Ant Design 的组件组合使用，提供良好的用户体验</font>
    - **<font style="color:rgb(51, 51, 51);">同步和异步验证</font>**<font style="color:rgb(51, 51, 51);">：支持多种验证规则和反馈机制，验证功能强大且易于配置</font>
    - **<font style="color:rgb(51, 51, 51);">表单布局</font>**<font style="color:rgb(51, 51, 51);">：提供多种布局选项，使表单易于设计和使用</font>

| **<font style="color:rgb(51, 51, 51);">特性</font>** | **<font style="color:rgb(51, 51, 51);">Formily</font>** | **<font style="color:rgb(51, 51, 51);">Ant Design Form</font>** |
| :--- | :--- | :--- |
| <font style="color:rgb(51, 51, 51);">表单状态管理</font> | <font style="color:rgb(51, 51, 51);">高度数据驱动，使用 Schema 管理表单结构和状态</font> | <font style="color:rgb(51, 51, 51);">基于组件的状态管理，使用 React Hooks (如 useForm)</font> |
| <font style="color:rgb(51, 51, 51);">数据绑定</font> | <font style="color:rgb(51, 51, 51);">强大的双向数据绑定与动态表单生成</font> | <font style="color:rgb(51, 51, 51);">单向数据流，依赖 React 的状态管理</font> |
| <font style="color:rgb(51, 51, 51);">动态表单生成</font> | <font style="color:rgb(51, 51, 51);">支持动态 Schema 配置，适合复杂和动态变化的表单</font> | <font style="color:rgb(51, 51, 51);">支持通过动态渲染组件来生成表单，但灵活性较低</font> |
| <font style="color:rgb(51, 51, 51);">表单初始化与重置</font> | <font style="color:rgb(51, 51, 51);">通过 Schema 和数据驱动进行初始化和重置</font> | <font style="color:rgb(51, 51, 51);">提供 initialValues 和 form.resetFields 方法</font> |


## 适用场景
<font style="color:rgb(51, 51, 51);">Formily 适用场景</font>

+ **<font style="color:rgb(51, 51, 51);">复杂和动态表单</font>**<font style="color:rgb(51, 51, 51);">：需要根据数据动态生成表单结构和字段的应用，如配置中心、CMS 系统等</font>
+ **<font style="color:rgb(51, 51, 51);">高可定制化需求</font>**<font style="color:rgb(51, 51, 51);">：需要高度自定义表单逻辑和验证规则的场景</font>
+ **<font style="color:rgb(51, 51, 51);">框架无关开发</font>**<font style="color:rgb(51, 51, 51);">：在多框架或混合框架项目中，需要统一的表单管理解决方案</font>
+ **<font style="color:rgb(51, 51, 51);">数据驱动开发</font>**<font style="color:rgb(51, 51, 51);">：需要严密的数据管理和数据驱动表单逻辑的场景</font>

<font style="color:rgb(51, 51, 51);">Ant Design Form 适用场景</font>

+ **<font style="color:rgb(51, 51, 51);">快速开发</font>**<font style="color:rgb(51, 51, 51);">：需要快速构建符合 Ant Design 规范的表单，提升开发效率</font>
+ **<font style="color:rgb(51, 51, 51);">标准化表单</font>**<font style="color:rgb(51, 51, 51);">：适用于标准和中等复杂度的表单，如登录、注册、用户设置等</font>
+ **<font style="color:rgb(51, 51, 51);">React 生态项目</font>**<font style="color:rgb(51, 51, 51);">：专为 React 设计，与 Ant Design 组件无缝集成的项目</font>
+ **<font style="color:rgb(51, 51, 51);">UI 一致性需求</font>**<font style="color:rgb(51, 51, 51);">：需要保证应用内表单的视觉一致性和用户体验</font>



