---
title: 深入解读pnpm patch packages的底层实现：实现自定义忽略文件或目录的方法探索
description: 在本文中，我们将深入讨论pnpm的patch-package功能，并探讨如何实现自定义忽略文件或目录的方法。
date: 2023-06-14
image: intro.jpg
categories: [ "前端", "pnpm"]
tags: [ "前端", "pnpm", "patch-package", "patch commit", "git", "diff"] 
---

# 概论

在本文中，我们将深入讨论pnpm的patch-package功能，并探讨如何实现自定义忽略文件或目录的方法。我们将回顾patch-package的原理，介绍Git中的diff命令以及它们在patch commit中的应用。然后，我们将讨论文件或目录忽略的重要性和应用场景，并探讨在patch commit中实现自定义忽略的新方法。

在实现自定义忽略文件或目录的方法方面，我们将详细讲解如何获取要打包的文件列表。

接下来，我们将解释在改动和实现过程中所做的具体改动和代码逻辑，并分析在patch commit中实现自定义忽略功能的关键步骤。

最后，我们将分享在实现过程中遇到的挑战和解决方案，并总结实现自定义忽略功能对于技术能力提升和知识拓展的影响。

通过本文的阅读，读者将对pnpm的patch-package功能有更深入的了解。

## 引言

### 介绍patch-package的概念和作用

在前端开发中，我们经常会遇到一些依赖包的问题，比如某个包有bug，或者某个包的功能不符合我们的需求。这时候，我们可能会想要修改这些包的源码，但是直接修改node_modules中的文件是不可取的，因为这样会导致依赖不稳定，而且每次安装或更新依赖都会覆盖我们的修改。那么，有没有一种方法可以让我们在不影响依赖管理的前提下，对某些包进行定制化的修改呢？答案是有的，那就是patch-package。

patch-package是一个npm包，它可以让我们对node_modules中的任何包进行修改，并且将这些修改保存在一个patch文件中。这样，我们就可以在项目中使用这个patch文件来覆盖原始的包，从而实现对依赖包的定制化。patch-package的使用非常简单，只需要安装它，然后在package.json中添加一个postinstall脚本，就可以在每次安装或更新依赖后自动应用patch文件。然而遗憾的是，在使用`pnpm`的情况下，这个包无法正常使用，但是pnpm官方新增了两个命令来处理这个问题：`pnpm patch xxx@xxx (--edit-dir xxx)`和`pnpm patch-commit (--edit-dir)`

### 为什么需要实现自定义忽略文件或目录的功能

pnpm patch-package是一个非常实用和强大的工具，它可以帮助我们解决很多依赖包的问题。但是，在使用它的过程中，我发现了一个问题：它没有提供一种方法来让我们自定义忽略某些文件或目录。这意味着，如果我们对一个包进行了修改，但是只想保留其中一部分修改，而忽略其他部分，那么我们就无法实现。比如IDE的配置文件、临时文件夹等。这样就会导致patch文件过大，而且可能会引入一些不必要或错误的修改。

但是`pnpm` **v8.6.2**之前的版本的patch-package并没有这个功能，于是笔者在pnpm的Git仓库提了一个issue：[Issues · pnpm/pnpm (github.com)](https://github.com/pnpm/pnpm/issues/6565)。
为了解决这个问题，我决定探索一种新的方法来实现自定义忽略文件或目录的功能。在本文中，我将详细介绍我所做的尝试和实现过程，并分享我在这个过程中的心得和收获。

## patch-package的原理回顾

在介绍我的方法之前，让我们先回顾一下patch-package的基本原理和流程。patch-package的核心思想是利用Git中的diff命令来生成和应用patch文件。diff命令可以比较两个文件或目录之间的差异，并以一种特定的格式输出结果。这种格式被称为unified diff format（统一差异格式），它可以用来描述两个版本之间的变化。例如：
```
diff
--- a/file1.txt
+++ b/file1.txt
@@ -1,3 +1,4 @@
+This is an important line
This is the original file
-This is a line to delete
+This is a modified line
This is another line
```

上面的例子展示了file1.txt文件从a版本到b版本之间的变化。其中：

- `---`和`+++`表示两个版本的文件名。
- `@@`表示变化发生的位置和范围。
- `+`表示新增的行。
- `-`表示删除的行。

有了这种格式，我们就可以用Git中的apply命令来将这些变化应用到另一个文件或目录上。例如：

```bash
git apply file1.patch
```

上面的命令会将file1.patch文件中的变化应用到当前目录下的file1.txt文件上。

patch-package就是利用了这种机制来实现对依赖包的修改。它的工作流程大致如下：

1. 在项目中安装或更新依赖包。
2. 在node_modules中找到要修改的包，并对其进行修改。
3. 运行`npx patch-package <package-name>`命令，生成一个patch文件，并保存在patches目录下。
4. 在package.json中添加一个postinstall脚本，如`"postinstall": "patch-package"`。
5. 在项目中使用patch文件覆盖原始的包。

这样，每次安装或更新依赖后，patch-package就会自动应用patch文件，从而实现对依赖包的定制化。

## 分析需求和实现思路

在探索如何实现自定义忽略文件或目录的功能之前，我们需要先分析一下这个需求的实现思路。最直观的想法是，我们需要一个配置文件来指定哪些文件或目录需要忽略。然后，在应用patch文件时，我们可以读取这个配置文件，并在应用补丁之前先忽略这些文件或目录。

现在我们需要考虑的就是如何实现这个配置文件。一个简单的方法是使用.gitignore配置文件，其中包含一个列表，列出了我们要忽略的文件或目录。但是，这样子看上去并不是很优雅和灵活，因为.gitignore配置文件主要是为git版本控制服务而设计的，而不是为补丁工具服务。在pnpm作者给出的建议下，我们可以利用现有的设置来指定哪些文件在打包时需要包含。这就需要使用package.json文件中的files字段。

我们前面提到，patch-package的核心思想是利用Git中的diff命令来生成和应用patch文件，所以分析得出，想要实现自定义忽略文件或目录的功能，我们需要在patchd操作的diff部分的代码中进行修改。具体来说，我们需要在进行diff操作之前先将需要忽略的文件过滤掉，或者在进行diff操作时对这些文件进行过滤。这样就可以确保在生成patch文件时排除这些文件，从而达到忽略文件或目录的目的。

## package.json文件中的files字段。

当我们在编写npm包时，可以在package.json文件中添加一个名为files的字段，用来指定**包含**在npm包中的文件和目录。这个字段的值是一个数组，其中列出的文件或目录将被包含在npm包中，而其他文件和目录则会被忽略。

例如，下面是一个简单的package.json文件，其中包含了files字段：

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "description": "My awesome package",
  "files": [
    "src",
    "index.js",
    "LICENSE"
  ]
}
```

上面的例子中，我们列出了包中包含的文件和目录，包括src目录、index.js文件和LICENSE文件。这意味着，当我们发布这个包时，只有这些文件会被包含在npm包中。

### 介绍Git中的diff命令以及它们在pnpm patch commit中的应用

有了前面的分析，我很快就找到了patchd操作的diff部分,其核心源码如下所示

```ts
async function diffFolders (folderA: string, folderB: string) {
  const folderAN = folderA.replace(/\\/g, '/')
  const folderBN = folderB.replace(/\\/g, '/')
  let stdout!: string
  let stderr!: string

  try {
    const result = await execa('git', ['-c', 'core.safecrlf=false', 'diff', '--src-prefix=a/', '--dst-prefix=b/', '--ignore-cr-at-eol', '--irreversible-delete', '--full-index', '--no-index', '--text', folderAN, folderBN], {
      cwd: process.cwd(),
      env: {
        ...process.env,
        GIT_CONFIG_NOSYSTEM: '1',
        HOME: '',
        XDG_CONFIG_HOME: '',
        USERPROFILE: '',
        // #endregion
      },
    })
    stdout = result.stdout
    stderr = result.stderr
  } catch (err: any) {
    stdout = err.stdout
    stderr = err.stderr
  }
  // we cannot rely on exit code, because --no-index implies --exit-code
  // i.e. git diff will exit with 1 if there were differences
  if (stderr.length > 0)
    throw new Error(`Unable to diff directories. Make sure you have a recent version of 'git' available in PATH.\nThe following error was reported by 'git':\n${stderr}`)

  return stdout
    .replace(new RegExp(`(a|b)(${escapeStringRegexp(`/${removeTrailingAndLeadingSlash(folderAN)}/`)})`, 'g'), '$1/')
    .replace(new RegExp(`(a|b)${escapeStringRegexp(`/${removeTrailingAndLeadingSlash(folderBN)}/`)}`, 'g'), '$1/')
    .replace(new RegExp(escapeStringRegexp(`${folderAN}/`), 'g'), '')
    .replace(new RegExp(escapeStringRegexp(`${folderBN}/`), 'g'), '')
    .replace(/\n\\ No newline at end of file$/, '')
}
```

在上述代码中，有一个使用了`git diff`命令的函数`diffFolders`，它用于比较两个文件夹之间的差异，并返回一个patch文件的内容。

在`diffFolders`函数中，我们使用了`git diff`命令来比较两个文件夹之间的差异。其中，`--src-prefix=a/`和`--dst-prefix=b/`选项指定了差异内容的前缀，`--ignore-cr-at-eol`选项用于忽略行结束符的不同，`--irreversible-delete`选项用于禁止逆向删除（即文件从B目录移动到A目录）的操作，`--full-index`选项用于生成完整的Git索引，`--no-index`选项用于指定不使用Git索引进行比较，`--text`选项用于指定比较的内容为文本文件。最后，通过正则表达式的替换操作，将差异内容中的前缀和文件路径进行了处理，去除了不必要的前缀信息。

`pnpm patch commit`命令用于提交由`pnpm patch`生成的patch文件，它可以接收一个文件夹路径作为参数，表示该文件夹中包含一个或多个由`pnpm patch`生成的patch文件。而`pnpm patch commit`命令就是利用了`diffFolders`函数帮助我们生成将要patch文件的内容。

## 实现自定义忽略文件或目录的方法

难题就差如何过滤掉配置的文件，而留下需要diff的文件列表了。我开始着手在patch-package中实现自定义忽略文件或目录功能。

### 方案一：手写忽略文件的逻辑

最初的想法是手写忽略文件的逻辑。在应用补丁前，读取package.json中的"files"字段，然后遍历需要打补丁的文件，将除了在"files"字段中列出的文件或目录进行忽略。这个实现起来比较简单，但需要用到一些node.js内置模块，如fs、path和glob。

首先，我们需要读取package.json中的"files"字段到一个数组中，接下来，我们遍历需要打补丁的文件，使用node.js的glob模块来查找包括子文件夹在内的所有文件。然后，再将文件路径进行拆分，与文件列表中的文件或目录进行比较，如果路径与忽略列表中的文件或目录匹配，则将该文件从处理列表中拷贝到一个临时文件夹。这样子实现的效果是可以过滤掉需要忽略的文件或目录，但是这个实现很不完美，会漏掉一些文件，也会包含一些不必要的文件，还存在一些潜在的Bug。这是因为手写过滤代码可能存在很多细节问题，而且也不够灵活。

### 方案二：使用npm-packlist

为了解决手写过滤代码会存在的问题，我开始寻找其他解决方案。最终我找到了一个名为npm-packlist的npm包

## npm-packlist包的介绍和使用和优势

npm-packlist是一个npm的工具包，它可以帮助我们在打包一个npm包的时候，自动获取要添加到包中的文件列表。这样我们就不用手动指定哪些文件是需要打包的，哪些文件是可以忽略的，从而提高了打包的效率和准确性。

npm-packlist的优势在于它遵循了一些规则，来自动过滤掉一些不必要或者敏感的文件，比如：

- 如果有package.json文件，并且有files字段，那么只打包files字段指定的文件。
- 总是打包readme, license, licence, copying, notice和package.json文件，如果它们存在的话。
-  如果没有package.json文件或者没有files字段，并且有.npmignore文件，那么忽略.npmignore文件指定的文件。
- 如果没有package.json文件或者没有files字段，并且没有.npmignore文件，但是有.gitignore文件，那么忽略.gitignore文件指定的文件。
- 忽略根目录下的node_modules文件夹，除非它是一个捆绑依赖。
- 忽略一些常见的无用或者危险的文件，比如.npmrc, .gitignore, .npmignore, .DS_Store, npm-debug.log等。

### 利用npm-packlist包获取要打包的文件列表之后的实现

总的来说，实现自定义忽略文件或目录的功能可以基于npm-packlist包实现。具体步骤如下：

1. 读取package.json中的"files"字段
2. 使用npm-packlist包过滤需要打补丁的文件和目录。
3. 将npm-packlist包过滤的结果拷贝到临时文件夹。
3. 直接将临时文件夹与源码目录进行diff比较。
3. 完活了

这种实现方法可以很好地避免手写过滤代码时存在的问题，不仅更加灵活，而且更加易于维护和扩展

我的实现代码如下：

```ts
/**
 * pnpm patch commit处理安装命令的处理程序
 * @param opts 安装命令的选项，包括 install.InstallCommandOptions 和 Config 的部分属性
 * @param params 参数列表，包含用户目录路径
 * @returns Promise，安装操作的结果
 */
export async function handler(
  opts: install.InstallCommandOptions & Pick<Config, 'patchesDir' | 'rootProjectManifest'>,
  params: string[]
) {
  // 获取用户变更代码目录，也就是pnpm patch commit命令的第一个参数
  const userDir = params[0]
  // 获取lockfile文件目录
  const lockfileDir = opts.lockfileDir ?? opts.dir ?? process.cwd()
  // 获取补丁目录名称
  const patchesDirName = normalizePath(path.normalize(opts.patchesDir ?? 'patches'))
  // 拼接补丁目录路径
  const patchesDir = path.join(lockfileDir, patchesDirName)
  // 创建补丁目录
  await fs.promises.mkdir(patchesDir, { recursive: true })
  // 从用户目录中读取补丁后的 package.json 文件
  const patchedPkgManifest = await readPackageJsonFromDir(userDir)
  // 获取补丁后的包名称和版本号
  const pkgNameAndVersion = `${patchedPkgManifest.name}@${patchedPkgManifest.version}`
  // 创建临时目录
  const srcDir = tempy.directory()
  // 将补丁后的包写入临时目录
  await writePackage(parseWantedDependency(pkgNameAndVersion), srcDir, opts)

  // 过滤并复制文件到临时目录
  const filteredFolder = await filterAndCopyFiles(userDir, tempy.directory())
  // 获取补丁内容
  const patchContent = await diffFolders(srcDir, filteredFolder)

  // 根据包名称和版本号生成补丁文件名
  const patchFileName = pkgNameAndVersion.replace('/', '__')
  // 将补丁内容写入补丁文件
  await fs.promises.writeFile(path.join(patchesDir, `${patchFileName}.patch`), patchContent, 'utf8')
  
  // 尝试读取项目清单文件和根项目package.json文件
  const { writeProjectManifest, manifest } = await tryReadProjectManifest(lockfileDir)
  const rootProjectManifest = opts.rootProjectManifest ?? manifest ?? {}

  // 如果项目清单中没有 pnpm 属性，则创建该属性
  if (!rootProjectManifest.pnpm) {
    rootProjectManifest.pnpm = {
      patchedDependencies: {},
    }
  }
  // 如果项目清单中的 pnpm 属性没有 patchedDependencies 属性，则创建该属性
  else if (!rootProjectManifest.pnpm.patchedDependencies) {
    rootProjectManifest.pnpm.patchedDependencies = {}
  }
  // 将补丁路径添加到根项目清单的 patchedDependencies 中
  rootProjectManifest.pnpm.patchedDependencies![pkgNameAndVersion] = `${patchesDirName}/${patchFileName}.patch`
  // 将更新后的根项目清单写入文件
  await writeProjectManifest(rootProjectManifest)

  // 如果存在选定的项目图谱，并且锁定文件目录存在于图谱中，则更新对应项目的清单
  if (opts?.selectedProjectsGraph?.[lockfileDir]) {
    opts.selectedProjectsGraph[lockfileDir].package.manifest = rootProjectManifest
  }

  // 如果存在所有项目的图谱，并且锁定文件目录存在于图谱中，则更新对应项目的清单
  if (opts?.allProjectsGraph?.[lockfileDir].package.manifest) {
    opts.allProjectsGraph[lockfileDir].package.manifest = rootProjectManifest
  }

  // 调用 install.handler() 完成安装操作
  return install.handler({
    ...opts,
    rawLocalConfig: {
      ...opts.rawLocalConfig,
      'frozen-lockfile': false,
    },
  })
}

async function filterAndCopyFiles(source: string, destination: string) {
  // 利用packlist获取源目录下的所有文件列表
  const files = await packlist({ path: source })
  // 如果文件列表为空，则返回源目录
  if (files.length === 0) {
    return source
  }
  // 复制文件到目标目录
  await Promise.all(
    files.map(async (file) => {
      const sourcePath = path.join(source, file)
      const destinationPath = path.join(destination, file)
      const destDir = path.dirname(destinationPath)
      await fs.promises.mkdir(destDir, { recursive: true })
      await fs.promises.copyFile(sourcePath, destinationPath)
    })
  )

  return destination
}

```

最终提交到官方Git仓库还做了一些其他的改动，但是总体的思路不变。感兴趣可以去查看具体的pr结果。

## 结论

通过上述实现步骤，我们成功地实现了自定义忽略文件或目录的功能，可以让我们更加方便地修改依赖包而避免不必要或错误的修改。需要注意的是，我们并没有修改pnpm patch-package本身的代码，而是在现有的设置基础上，增加了一个自定义忽略列表。这样既不会影响原来的功能，也不会引入新的依赖或风险。

当然，这种方法还有一些局限性。例如，它只适用于使用files字段的项目。并且需要编写大量的文件列表。期待有更好的方案。

## 参考资料

- [pnpm patch-package官方文档](https://pnpm.io/zh/cli/patch-commit)
- [How to exclude files from diff at pnpm patch #6565](https://github.com/pnpm/pnpm/issues/6565)
- [unified diff format](https://en.wikipedia.org/wiki/Diff#Unified_format)
- [Git diff documentation](https://git-scm.com/docs/git-diff)
- [patch-package repository](https://github.com/ds300/patch-package)
