🔨 测试相关文章：
1. [⭐️⭐️ The little guide to CI/CD for frontend developers - maximeheckel](https://blog.maximeheckel.com/posts/guide-to-cicd-for-frontend-developers/) 虽然是讲CI/CD，但是主要内容还是测试，以React项目为例子
   - 先大致介绍了CI/CD的主要流程：
     - `Linting/Formatting`: ESLint + Prettier 校验和格式化
     - `Unit tests` + `Integration tests` + `E2E tests`： 各种测试，重点讲解内容
     - 构建分发：已 Github Actions 为例
   - 单元测试：针对 **组件 + Reducers/State/Actions + 工具函数**
     - 可能用到的库 `jest` + `@testing-library/react` + `@testing-library/react-hooks` + `@testing-library/jest-dom`
   - 集成测试：针对 **Navigation + Forms + Views** 本地mock，各种可能的情形，比如 **成功有数据 | 成功没数据 | 失败错误**
     - `jest` + `@testing-library/react` + `Cypress` + [msw](https://github.com/mswjs/msw)
   - 端对端测试：这个是模拟真实环境，涉及到服务器，**可能是最慢的， 只针对 `成功` 的情形进行测试**
   - Accessibility Tests: 这个一般国外用的比较多，可能使用到的库
     - `Lighthouse CI` + `Cypress Axe`
     - 浏览器工具： `Accessibility Insight` + `Chrome Lens`
   - 自动化流水线： 针对 `Pull-Request` 进行持续集成，首先先跑 `Linting + Formatting`; 然后跑上面介绍的各种测试；最后发布阶段，添加一个 `production` 分支，推送到该分支时，自动进行发布操作，并且可以添加预览地址
