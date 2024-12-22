<font style="color:rgb(43, 43, 43);">Monorepo 是一种项目代码管理方式，指单个仓库中管理多个项目，有助于简化代码共享、版本控制、构建和部署等方面的复杂性，并提供更好的可重用性和协作性。</font>

## <font style="color:rgb(43, 43, 43);">优缺点</font>
### 优点
1. 代码重用和共享：不同的项目可以共享代码库，减少重复代码。
2. 一致的依赖和配置：可以更容易地管理依赖库版本和共享配置，确保各个项目保持一致。
3. 简化 CI/CD：统一的代码库可以简化持续集成和部署流程。
4. 原子性变更：在需要同时修改多个项目时，可以在一个提交中完成所有变更，确保一致性。

### 缺点
1. 仓库规模大：随着项目增长，仓库体积可能会显著增加，影响性能。
2. 复杂的权限管理：可能需要额外的措施来管理不同项目的访问权限。
3. 复杂的构建和测试流程：可能需要更复杂的构建和测试工具来管理多个项目。

## lerna
```plain
my-monorepo/
├── packages/
│   ├── package-a/
│   │   ├── src/
│   │   ├── package.json
│   │   ├── index.js
│   ├── package-b/
│   │   ├── src/
│   │   ├── package.json
│   │   ├── index.js
├── package.json
├── lerna.json
```

### 常用命令
1. 安装与初始化：通过 lerna init 初始化项目。
2. 添加和管理包：通过 lerna create 和 lerna add 添加和管理包依赖。
3. 构建与测试：通过 lerna run 命令统一管理构建和测试。
4. 版本控制与发布：通过 lerna version 和 lerna publish 管理版本和发布流程。

### 共享依赖
在 Monorepo 项目中，通常会希望共享依赖库，例如 ESLint、Babel 等。可以在 Monorepo 根目录的 package.json 中安装这些共享依赖

### 版本管理
Lerna 提供了两种版本管理策略：固定版本（Fixed/Locked mode）和独立版本（Independent mode）。

+ 固定版本：所有包共享相同的版本号
+ 独立版本：每个包可以有独立的版本号

