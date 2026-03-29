# 步骤 4：还原 UI 设计

从前端代码中还原 UI/UX 设计。

**分析内容：**

1. **页面清单**
   - 从路由文件列出所有页面
   - 每个页面的用途（从组件内容推断）
   - 页面间的导航关系

2. **信息架构**
   - 从导航组件还原页面层级
   - 从路由嵌套关系还原页面结构

3. **页面布局**
   - 从布局组件还原页面区域划分
   - 从组件组合还原页面内容结构

4. **用户流程**
   - 从表单提交/按钮事件还原操作流程
   - 从路由跳转还原页面间流转

5. **状态处理**
   - 搜索 loading/error/empty 状态的处理代码
   - 标注哪些页面有完整的状态处理，哪些缺失

6. **视觉规范提取（L3）**
   - 从 CSS 变量/tailwind.config/theme 文件中提取：
     - 色彩系统：primary、secondary、accent、semantic 色值
     - 排版：字体族、字号层级（搜索 text-xs/sm/base/lg/xl 或 font-size 定义）
     - 间距：基础单位（搜索 spacing/gap/padding 的常用值）
     - 圆角：搜索 border-radius/rounded 的常用值
     - 阴影：搜索 box-shadow/shadow 定义
   - 如果使用了 UI 框架（shadcn/MUI/Ant），标注框架名和主题自定义程度

7. **动效规范提取（L4）**
   - 搜索 transition/animation/keyframes 相关代码
   - 提取：常用的 duration（150ms/300ms/500ms）、easing（ease-out/ease-in-out）
   - 识别加载模式：骨架屏(Skeleton)？旋转器(Spinner)？渐进式？
   - 识别页面过渡：有没有 framer-motion/react-transition-group 等动画库？
   - 如果没有发现动效代码，标注"项目未定义动效规范"

**跳过条件：** 如果项目没有前端（纯后端/CLI 工具），跳过此步骤。
