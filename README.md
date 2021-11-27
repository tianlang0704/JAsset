# JAsset

JAsset是基于XAsset4.0魔改实现的Unity热更新资源管理插件，将用于[JEngine](https://github.com/JasonXuDeveloper/JEngine)框架

### 开发目标

| 功能                                                         | 完成时间       | 发布时间       |
| ------------------------------------------------------------ | -------------- | -------------- |
| 将UnityWebRequest换为System.Net的HttpCleint或HttpWebRequest  | -              |                |
| 实现分包功能，可以按需下载                                   | 2021年11月26日 | 2021年12月10日 |
| 修改配置面板，使其支持分包                                   | 2021年11月26日 | 2021年12月10日 |
| 自定义特定资源是否打ab，不打的话也可以通过该接口使用指定资源 | -              |                |
| 修复切后台导致的下载终止                                     | -              |                |
| 修复下载速度为0的概率问题                                    | -              |                |

### 使用

1. 场景内拼个UI界面挂UpdateScreen（参考JEngine的Init场景），这个是更新时的UI控件
2. 创建个Updater
3. 注册一下Updater.OnAssetsInitialized，即当前包的下载回调，回调有2个参数，第一个是跳转带后缀的场景名，第二个是跳转场景的进度（如果有跳转）（这块参考JEngine的InitJEngine.cs）
4. 需要手动注册可寻址路径（参考JEngine的InitJEngine.cs）
5. 跳转场景的路径最好写完整路径，或从分包目录开始的短路径（例如我这个资源在Assets/HotUpdateResources/AddOns/AddOn1/Scenes/test.unity，我可以写AddOn1/Scenes/test.unity）
6. 资源加载可以用```Assets.LoadAsset(path, type, package)```，第一个参数是路径，建议路径和上面说的一样，type是资源类型，一般是UnityEngine.Object，文件就是TextAsset，音频可以是AudioClip，其他自行参考，packet是分包名，主包资源这里留空或写Main都可以，分包资源这里必填
7. 启动热更，给gameObject挂个Updater配置baseurl（资源下载地址），然后GameScene（跳转场景，带后缀），MainPackageName（首包名称），Developement（开发模式，无需下载AB），Enable VFS（是否启用vfs，可选，自行了解vfs是啥）
8. 打AB调```BuildScript.BuildAssetBundles();```即可
9. 如果不清楚咋用资源加载的API，参考JEngine的AssetMgr.cs
10. 如果不清楚JEngine的参考代码在哪，去JEngine仓库搜索即可



### 分包模块

#### 配置

- 创建多个Rules（在ScruptableObjects目录下），在Rules内配每个包的包名（全英文）和资源（场景单独放一个目录，然后用Path模式，即每个场景单独打AB包）
- 保持JEngine框架内的目录格式，即确保分包资源在HotUpdateResources/AddOns目录下，其他主包资源在HotUpdateResources其他目录下
- 确保有一个主包的Rules配置，同时Updater的MainPackageName填写对应的主包名称
- 如果有热更代码，建议把热更代码放主包，没有可以忽略

#### 代码相关

- 需要手动注册可寻址路径：```Assets.AddSearchPath("Assets/HotUpdateResources/AddOns")```

- 下载指定包：

  ```c#
  Updater.UpdatePackage("AddOn1", "AddOn1/Scenes/test.unity", () =>
                        {
                          Debug.Log("场景已切换");
                        }, () =>
                        {
                          Debug.Log("下载已完成");
                          var res = JResource.LoadRes<TextAsset>("AddOn1/Others/test.txt", "AddOn1");
                          Debug.Log($"文件内容：{res.text}");
                        }, ()=>
                        {
                          canvas.gameObject.SetActive(false);
                          Debug.LogError("下载被取消");
                        },canvas);
  ```

  第一个参数是Rules内配的包名，后面参数都可为空，第二个参数是跳转场景，第三个参数场景切换后回调，第四个参数下载完成后回调（如果有跳转场景回调，会在其之后调用），第四个参数是下载去写回调，第五个参数是UI（UpdateScreen）

- 删除指定包可以用：```Updater.ClearPackage(packageName)```，留空即为删除主包
