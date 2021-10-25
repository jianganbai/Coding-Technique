# 软件工程相关

## build

- 小概念

  - build: 执行很多指令以搭建程序等
  - dependency: 为了成功build，需要先安装的辅助包?
  - rule：如何用dependency搭建程序

- make

  -  make指令默认找当前目录下的Makefile文件以进行build

  - ```makefile
    # Makefile例子
    paper.pdf: paper.tex plot-data.png  # paper.pdf需要2个依赖
    	pdflatex paper.tex  # 满足依赖后执行该命令
    	
    plot-%.png: %.dat  # %代表任意字符串，2个%对应相同的字符串
    	./plot.py -i $*.dat -0 $@  # 必须使用Tab
    ```
    
  -  cmake比make更适合大型C程序

## Version

- 版本号：major version . minor version . patch version
  - 更新高级version，低级version全部从0开始
  - 一般仅patch不同，是兼容的

## CI/CD

- 持续集成：Continuous Integration
  - 提交代码后就对代码进行测试
  - github action：给定代码提交流程？
  - github pages：将markdown转化为网页并发布
- 持续交付
  - 测试通过的新代码部署到类生产环境中（staging）
  - 再手动将代码部署到生产环境中
- 持续部署：Continuous Deployment
  - 自动将测试通过的代码部署到生产环境中