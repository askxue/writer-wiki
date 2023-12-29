# Writerside使用手册

## 插件

> 一些好用的插件

|         插件名称          |       说明        |
|:---------------------:|:---------------:|
|  Atom Material Icons  |      图标美化       |
|        .ignore        |    版本控制文件忽略     |
| Chinese Language Pack |      中文语言包      |
|    CodeGlance Pro     |     文件代码缩略图     |
|   Material Theme UI   |      UI美化       |
|  String Manipulation  | 字符大小写、驼峰、等等格式转换 |
|      Translation      |       翻译        |
|     Power Mode 3      | 敲代码的时候增加一些动画特效  |

## 发布到GitHub Pages

[Jetbrains官网文档](https://www.jetbrains.com/help/writerside//deploy-docs-to-github-pages.html)

1. 切换到 Settings|Pages 选项卡
2. Build and deployment 选项，Source选择 **GitHub Actions**

![image_1.png](image_1.png)

3. 在文档集根目录创建部署脚本 `.github/workflows/deploy.yml`

```yaml
name: Build documentation

on:
  # If specified, the workflow will be triggered automatically once you push to the `main` branch.
  # Replace `main` with your branch’s name
  push:
    branches: [ "main" ]
  # Specify to run a workflow manually from the Actions tab on GitHub
  workflow_dispatch:

# Gives the workflow permissions to clone the repo and create a page deployment
permissions:
  id-token: write
  pages: write

env:
  # Name of module and id separated by a slash
  INSTANCE: Writerside/hi
  # Replace HI with the ID of the instance in capital letters
  ARTIFACT: webHelpHI2-all.zip
  # Writerside docker image version
  DOCKER_VERSION: 232.10275
  # Add the variable below to upload Algolia indexes
  # Replace HI with the ID of the instance in capital letters
  ALGOLIA_ARTIFACT: algolia-indexes-HI.zip

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Build Writerside docs using Docker
        uses: JetBrains/writerside-github-action@v4
        with:
          instance: ${{ env.INSTANCE }}
          artifact: ${{ env.ARTIFACT }}
          docker-version: ${{ env.DOCKER_VERSION }}

      - name: Upload documentation
        uses: actions/upload-artifact@v3
        with:
          name: docs
          path: |
            artifacts/${{ env.ARTIFACT }}
            artifacts/report.json
          retention-days: 7

      # Add the step below to upload Algolia indexes
      - name: Upload algolia-indexes
        uses: actions/upload-artifact@v3
        with:
          name: algolia-indexes
          path: artifacts/${{ env.ALGOLIA_ARTIFACT }}
          retention-days: 7

  # Add the job below and artifacts/report.json on Upload documentation step above if you want to fail the build when documentation contains errors
  test:
    # Requires build job results
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v1
        with:
          name: docs
          path: artifacts

      - name: Test documentation
        uses: JetBrains/writerside-checker-action@v1
        with:
          instance: ${{ env.INSTANCE }}

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    # Requires the build job results
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: docs

      - name: Unzip artifact
        run: unzip -O UTF-8 -qq ${{ env.ARTIFACT }} -d dir

      - name: Setup Pages
        uses: actions/configure-pages@v2

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: dir

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1
```

> 如果您希望在文档自动测试未通过时构建失败，请添加test和工件 report.json，如上面的示例所示。
>
> 否则，即使自动测试失败，我们也会生成文档工件，并且错误将显示在构建日志中。

4. 在 _writerside.cfg_ 文件中设置 `web-path` 参数，声明 images 的仓库名称。

```xml

<ihp version="2.0">
  <topics dir="topics"/>
  <images dir="images" web-path="github-repository-name"/>
  <instance src="hi.tree"/>
</ihp>
```

5. 设置正确的环境变量

![image_3.png](image_3.png)

> 需配置正确的docker image版本。  [更新log](https://plugins.jetbrains.com/plugin/20158-writerside/docs/2023.11.232.10275.html)
> {style="note"}

