> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7433805918494916618)

### 深入解析 effet.js：人脸识别、添加、打卡与睡眠检测的项目结构揭秘

近年来，面部识别和 AR 特效技术的普及让我们在日常应用中越来越多地接触到有趣的互动体验。而基于 facemesh.js 的二次开发框架——**effet.js**，则为开发者提供了一种简单而强大的方式来实现面部特效和识别功能。今天，我们将通过 effet.js 的项目结构，深入探讨其运行原理，看看它是如何利用前沿技术来实现流畅的人脸检测与交互的。

#### 1. effet.js 的整体架构

effet.js 的项目结构采用模块化设计，通过明确的目录划分，将各项功能进行清晰地组织和封装。我们可以将项目大致分为以下几个部分：

*   **components**：主要包括内部初始化数据、公共枚举以及管理当前 DOM 实例的逻辑。
*   **core**：核心模块，包含动作处理、数据库交互、DOM 操作等多项功能，是整个框架的关键部分。
*   **styles**：存放框架的样式文件，用于定义人脸特效的视觉表现。
*   **util**：各种通用工具函数，包括摄像头操作、条件检测、绘制等常用的辅助功能。
*   **index.js**：整个项目的入口文件，负责初始化和启动框架。

接下来，我们将详细介绍这些模块的作用及其在 effet.js 运行过程中的角色。

#### 2. components 组件模块

`components` 目录主要用于存放框架的公共组件和初始化数据。

*   **AppState.ts**：管理内部的初始化数据，包括摄像头、DOM 元素等基本信息。
*   **FaceManager.ts**：用于管理当前 DOM 单例，这个类的作用是确保在处理消息替换时，始终对同一 DOM 元素进行操作，避免出现不必要的内存占用和资源浪费。

这种设计让 effet.js 在处理多次人脸检测和特效应用时能够高效、稳定地管理 DOM 元素。

#### 3. core 核心模块

`core` 目录是 effet.js 的核心逻辑所在，涵盖了以下几个重要部分：

*   **action**：动作处理模块，包含人脸添加、登录检测、睡眠检测和打卡等功能。例如，`addFace/index.js` 负责处理用户人脸添加的逻辑，而 `checkLogin/index.js` 则用于人脸登录的检测。
*   **before**：动作前的预处理模块。每个动作在执行前，可能需要一些额外的检查或准备工作。例如，`checkLogin/index.js` 用于处理登录前的检查逻辑。
*   **db**：数据库模块，`db.js` 负责使用 IndexedDB 来存储和缓存用户数据、模型信息等。这种设计让 effet.js 可以离线运行，提升了用户体验。
*   **defaultAssign**：默认配置和参数分配模块，`assign.js` 用于为框架中的各个组件提供初始参数和默认值，确保框架在各种环境下均能正常运行。
*   **dom**：DOM 操作模块，包含创建和管理人脸相关 DOM 元素的逻辑，例如 `createFaceElements.js` 用于动态创建人脸特效所需的 DOM 元素。
*   **log**：用于屏蔽插件相关的内部警告信息，`log.js` 确保了控制台的整洁，方便开发人员进行调试。

核心模块的分层设计使得每个功能都具有独立性和可维护性，这对于复杂交互和特效的实现尤为重要。

##### 代码示例：人脸添加动作

以下是处理人脸添加动作的代码示例：

```
import { addFace } from './core/action/faceAction';

// 执行人脸添加逻辑
addFace().then(() => {
  console.log('人脸添加成功');
}).catch(error => {
  console.error('人脸添加失败:', error);
});


```

上述代码展示了如何调用核心模块中的人脸添加逻辑来实现用户的人脸注册功能。

#### 4. styles 样式模块

`styles` 目录包含所有与视觉表现相关的文件，用于定义人脸特效的外观。

*   **template**：存放不同功能的样式模板，如 `addFace` 和 `checkLogin` 等模块中的 `index.css` 定义了对应特效的样式。
*   **faceColor.js**：用于处理与人脸特效颜色相关的逻辑，让开发者可以根据需求自定义特效的颜色效果。
*   **root.css**：全局样式文件，定义了一些基础样式和布局。

样式模块确保了特效在浏览器中展示时具有一致性和视觉吸引力。通过将样式与逻辑分离，开发者可以更方便地对特效的外观进行调整。

##### 图片示例：人脸添加样式

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/c5f983e05ccc4780b5df159f135c11ce~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg55So5oi3NjExNDY2OTYxMjIw:q75.awebp?rk3s=f64ab15b&x-expires=1731463449&x-signature=lnZ4EI7myQ%2F6a913hYIVnPdCgbE%3D)

#### 5. util 通用工具模块

`util` 目录包含了多个工具函数，用于简化常见的任务，例如摄像头操作、距离计算和人脸网格处理。

*   **cameraAccessUtils.js** 和 **cameraUtils.js**：用于处理摄像头的访问和操作，如获取摄像头权限、切换摄像头等。
*   **faceMesh.js**：负责处理人脸网格的相关逻辑，例如通过 facemesh 模型获取人脸特征点。
*   **distanceUtils.js** 和 **drawingUtils.js**：用于进行距离计算和绘图操作，这些工具函数在渲染特效时被频繁使用。

这些工具函数的存在，让开发者可以专注于高层次的功能开发，而不必担心底层的细节实现。

##### 代码示例：摄像头访问工具

```
import { requestCameraAccess } from './util/cameraAccessUtils';

// 请求摄像头权限
requestCameraAccess().then(stream => {
  console.log('摄像头权限已获取');
  // 使用摄像头流进行进一步处理
}).catch(error => {
  console.error('无法获取摄像头权限:', error);
});


```

#### 6. 使用 IndexedDB 的缓存机制

effet.js 通过 `core/db/db.js` 使用 **IndexedDB** 来缓存模型和其他依赖资源，从而减少每次加载的时间。这种设计可以显著提升用户体验，尤其是对于网络环境不稳定的用户。

IndexedDB 是浏览器内置的一个低级 API，它允许我们在用户设备上存储大量结构化数据。在 effet.js 中，我们利用 IndexedDB 缓存 facemesh 模型和用户的面部数据，确保在用户首次加载模型后，后续访问时能够直接从缓存中读取数据，而无需重新下载模型。这不仅提升了加载速度，还降低了对网络的依赖。

##### 代码示例：使用 IndexedDB 缓存模型

```
// 缓存 facemesh 模型到 IndexedDB
async function cacheModel(modelData) {
  const db = await openIndexedDB('effetModelCache', 1);
  const transaction = db.transaction(['models'], 'readwrite');
  const store = transaction.objectStore('models');
  store.put(modelData, 'facemeshModel');
}

// 从 IndexedDB 加载模型
async function loadModelFromCache() {
  const db = await openIndexedDB('effetModelCache', 1);
  const transaction = db.transaction(['models'], 'readonly');
  const store = transaction.objectStore('models');
  return store.get('facemeshModel');
}

function openIndexedDB(dbName, version) {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open(dbName, version);
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}


```

通过这样的缓存机制，用户在第一次访问时可能需要加载较长时间，但后续的访问速度会大大提升，从而提供更流畅的用户体验。

#### 7. 核心模块中的预处理逻辑

在 `core/before` 目录下，包含了许多预处理逻辑，这些逻辑在每个动作执行之前运行，以确保后续的动作能够顺利进行。

*   **checkLogin/index.js**：负责在执行人脸登录之前检查用户的身份信息，例如判断用户是否已经注册或存在于数据库中。
*   **faceBefore.js**：是所有预处理逻辑的入口，通过调用不同的预处理函数，确保每个动作的执行环境和条件都已满足。

这种预处理机制有效地提高了系统的稳定性和安全性。例如，在进行人脸登录时，如果用户的数据尚未注册或缓存不完整，系统将通过预处理逻辑提前捕获这些问题并进行相应的处理。

##### 代码示例：登录前的预处理

```
import { checkLogin } from './core/before/faceBefore';

// 执行登录前的检查逻辑
checkLogin().then(() => {
  console.log('预处理完成，准备执行登录动作');
}).catch(error => {
  console.error('登录预处理失败:', error);
});


```

#### 8. DOM 操作和特效渲染

effet.js 中的 DOM 操作模块负责创建和管理人脸特效相关的 DOM 元素。通过对 DOM 的操作，我们可以将各种特效应用到用户的人脸上，并根据用户面部的变化来实时调整特效的位置和形状。

*   **createFaceElements.js**：用于动态创建用于渲染特效的 DOM 元素。这些元素包括虚拟的眼镜、面具等，通过将它们附加到特定的人脸特征点，可以实现各种视觉特效。
*   **defaultElement.js**：提供了一些默认的 DOM 元素配置，如特效的初始位置、大小和样式等。

##### 代码示例：创建人脸特效元素

```
import { createFaceElements } from './core/dom/createFaceElements';

// 创建用于渲染特效的 DOM 元素
createFaceElements().then(elements => {
  console.log('特效元素创建成功', elements);
}).catch(error => {
  console.error('特效元素创建失败:', error);
});


```

通过将特效元素与人脸特征点绑定，我们可以实现一些有趣的交互。例如，当用户张嘴或眨眼时，系统可以检测到这些动作并触发相应的视觉反馈，从而提升用户体验的互动性。

#### 9. 样式与用户体验

样式模块不仅仅是为了让特效看起来美观，更是为了确保其与用户的操作无缝对接。`styles/template` 目录下的样式模板被精心设计，以适应不同类型的设备和显示环境。

例如，`addFace/index.css` 和 `checkLogin/index.css` 分别定义了人脸添加和登录检测的样式，通过这些样式文件，开发者可以轻松地实现具有一致性且专业的用户界面。

##### 图片示例：多种特效风格

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/9fe13faf2e654b23823eb775cbcdf0a8~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg55So5oi3NjExNDY2OTYxMjIw:q75.awebp?rk3s=f64ab15b&x-expires=1731463449&x-signature=GJiqbXQu155nhp1pyBF%2FaaJQ%2F40%3D)

这种模块化的样式管理方式让我们可以快速调整不同特效的外观，同时确保代码的可读性和可维护性。在不断的版本迭代中，样式模块可以独立于逻辑模块进行修改，极大地提高了项目的可扩展性。

#### 10. effet.js 的初始化流程

`index.js` 作为项目的主要入口，负责初始化 effet.js 的所有模块。在初始化过程中，系统会调用核心模块、样式模块和工具函数，以确保整个框架能够无缝启动。

*   **initializeEffet**：这是一个主函数，负责加载配置、初始化摄像头、检测用户设备是否满足要求，并调用各个核心模块来启动面部检测和特效渲染。
*   **FaceManager**：用于管理初始化后的 DOM 元素，确保在不同的特效之间切换时，DOM 操作能够始终保持一致。

##### 代码示例：框架初始化

```
import { initializeEffet } from './core/index';

// 初始化 effet.js 框架
initializeEffet().then(() => {
  console.log('effet.js 初始化完成');
}).catch(error => {
  console.error('初始化失败:', error);
});


```

通过这样的初始化流程，我们可以确保框架的各个部分能够正确协同工作，从而为用户提供稳定且高质量的体验。

#### 11. 开发与优化的挑战

effet.js 在开发过程中遇到了许多挑战，例如：

*   **模型加载时间长**：由于 facemesh 模型文件较大，我们使用 IndexedDB 来缓存模型，减少用户每次访问的加载时间。
*   **资源丢包与网络不稳定**：为了解决网络环境不稳定的问题，我们采用了离线缓存策略，并通过优化加载顺序和减少请求次数来提升加载速度。
*   **性能优化**：为了保证在中低端设备上的流畅运行，我们对人脸检测和特效渲染进行了大量优化，包括减少计算开销、利用 WebGL 加速等。

这些挑战促使我们不断改进框架的架构和实现方式，以确保 effet.js 在不同的设备和网络环境下都能表现出色。

#### 结语

effet.js 是一个旨在降低面部识别和特效开发门槛的框架，通过模块化的项目结构封装复杂的底层逻辑，让开发者可以专注于创造有趣的互动体验。通过组件、核心模块、样式、工具函数以及缓存机制的有机结合，effet.js 为开发者提供了强大的基础设施来构建各种人脸交互应用。

通过这篇文章，我们展示了 effet.js 的整体架构、人脸检测与特征点定位、特效渲染、交互逻辑实现以及优化挑战，并结合代码示例和图片示例来帮助你更好地理解其运行原理。希望你能够从中获得启发，创造更多有趣的应用！

如果你有任何问题或者想要进一步了解它的应用，欢迎在评论区留言讨论！

这篇博客旨在详细介绍 effet.js 的运行原理和模块化设计，帮助开发者深入了解其工作机制。在未来的开发中，我们期待更多的人参与到这个项目中来，共同探索面部识别和特效技术的更多可能性。

更多资源
----

*   官方文档：[faceeffet.com](https://link.juejin.cn?target=https%3A%2F%2Ffaceeffet.com "https://faceeffet.com")
*   GitHub 仓库：[github.com/typsusan/ef…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ftypsusan%2Feffet "https://github.com/typsusan/effet")
*   Gitee 仓库：[gitee.com/susantyp/ef…](https://link.juejin.cn?target=https%3A%2F%2Fgitee.com%2Fsusantyp%2Feffet "https://gitee.com/susantyp/effet")