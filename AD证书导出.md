# AD证书导出

## 一、AD 证书安装过程

  环境说明：这里我用的服务器是 Windows server 2008 

​           *如果是第一次安装，为了避免出错，下列步骤中能勾选的复选框都选上。

​           *因服务器而异，在安装过程中，仔细阅读windows的说明信息，根据自己的需求作响应调整。

​     1.Windows server，进入服务器管理器，如图1.1所示。       ![ab52b2da7414b67beeb60322820b925b6ad15d97](https://yqfile.alicdn.com/ab52b2da7414b67beeb60322820b925b6ad15d97.jpeg)

​                                 图1 .1服务器管理器

​     2.服务器管理器，选择角色，右击，添加角色，如图1.2所示

​      ![ee4f22da7a5988920477c83511546497c6c40b27](https://yqfile.alicdn.com/ee4f22da7a5988920477c83511546497c6c40b27.jpeg)

​                                       图1.2 添加角色

​    3.添加角色第一步，选择下一步，如图3所示

​     ![085be6b307bc60710524942c3d0d3fce94cd19ca](https://yqfile.alicdn.com/085be6b307bc60710524942c3d0d3fce94cd19ca.jpeg)

​                                 图1.3 添加角色-第一步

​       4.添加角色第二步，选择Active Directory,下一步，如图4所示

​      ![fde550a617dfdfadb27e1ba2043ba699a09eb50e](https://yqfile.alicdn.com/fde550a617dfdfadb27e1ba2043ba699a09eb50e.jpeg)

​                                       图1.4 添加角色-第二步

​     5.添加角色第三步，选择证书颁发机构web注册，会弹出添加角色向导,点击添加必须的角色服务，（这里注意能选的都选上，或者仔细读win说明）如图1.5所示

​     ![8a1f3b04c9e4125a6e6a79d0f4caa1c39c23a5f9](https://yqfile.alicdn.com/8a1f3b04c9e4125a6e6a79d0f4caa1c39c23a5f9.jpeg)

​                             图1.5 添加角色-第三步

​    6.添加角色第四步，选择安装类型-企业，如图1.6所示

​     ![01632f4bb62a5cb5b4b5175602886b4b66015116](https://yqfile.alicdn.com/01632f4bb62a5cb5b4b5175602886b4b66015116.jpeg)

​                             图1.6 添加角色-第四步

​     7.添加角色第五步，选择CA类型-根CA，如图1.7所示

​     ![1400bd20ebba67a256bfae2cb970f160e479836f](https://yqfile.alicdn.com/1400bd20ebba67a256bfae2cb970f160e479836f.jpeg)

​                             图1.7 添加角色-第五步

​    8.添加角色第六步，选择新建私钥，如图1.8所示

​     ![70e51c6ea1b053fdf4d0a4b280e1b27c5a64ef35](https://yqfile.alicdn.com/70e51c6ea1b053fdf4d0a4b280e1b27c5a64ef35.jpeg)

​                             图1.8 添加角色-第六步

​     9.添加角色第七步，这里我用的是默认，密钥字体长度2048，哈希算法啥sha1，如图1.9所示

​     ![8c26e6f5a3cbaee42d924c7ba77726e2760aacc6](https://yqfile.alicdn.com/8c26e6f5a3cbaee42d924c7ba77726e2760aacc6.jpeg)

​                             图1.9 添加角色-第七步

​     10.添加角色第八步，输入CA及dc，如图1.10所示

​     ![0f4a91ca3140ea85e6b14d06f21b3cb03611266f](https://yqfile.alicdn.com/0f4a91ca3140ea85e6b14d06f21b3cb03611266f.jpeg)

​                             图1.10 添加角色-第八步

​     11.添加角色第九步，这里怕错的话能选的都选上，如图1.11所示

​     ![4dc1fef56ad8ce6010d28511de01c67d7c60ab8c](https://yqfile.alicdn.com/4dc1fef56ad8ce6010d28511de01c67d7c60ab8c.jpeg)

​                             图1.11添加角色-第九步

​     11.添加角色第十步，至此安装成功，如图1.12所示

​     ![fe0b1796e65c230f5e996ba2a6282c5b05c75518](https://yqfile.alicdn.com/fe0b1796e65c230f5e996ba2a6282c5b05c75518.jpeg)

​                             图1.12添加角色-第九步

## 二、AD证书导出

​     说明：这里将导出的AD证书导入到java密钥库中，根据实际情况灵活使用。

​           *如果是第一次安装，为了避免出错，把能勾选的复选框都选上。

​           *这里要导出俩个证书，域根证书和计算机证书

​      1.使用快捷键win+R进入‘运行’，输入mmc进入控制台，如图2.1。

​      ![286e023d473d9581f9e0552838287b45488e0c31](https://yqfile.alicdn.com/286e023d473d9581f9e0552838287b45488e0c31.jpeg)

​                            图2.1进入控制台

​     2.进入控制台，如果还没节点，如图2.2。可以添加，这里需要证书的节点。

​      ![529e88dfdcdf78ab70563434d9fa02ea81a3762d](https://yqfile.alicdn.com/529e88dfdcdf78ab70563434d9fa02ea81a3762d.jpeg)

​                            图2.2控制台-空

​     3.添加管理单元，右击文件，选择添加／删除管理单元（本地计算机），如图2.3所示。

​      ![2012f040e52b16ca23b22b56eca2f79f88b282d4](https://yqfile.alicdn.com/2012f040e52b16ca23b22b56eca2f79f88b282d4.jpeg)

​                            图2.3添加管理单元

​      4.管理单元，选择‘证书’节点，点击添加，确定，如图2.4所示。

​      ![1d9a22bc68b26b5f239122e305287536ca4b8b87](https://yqfile.alicdn.com/1d9a22bc68b26b5f239122e305287536ca4b8b87.jpeg)

​                            图2.4添加证书节点

​      5.添加完‘证书’节点后，打开‘个人’，‘证书’，选择要导出的证书，如图2.5所示。

​      ![3a258aa112ec1b2ad96b15820dc3e584d6ace73a](https://yqfile.alicdn.com/3a258aa112ec1b2ad96b15820dc3e584d6ace73a.jpeg)

​                            图2.5导出证书，第一步

​      6.右击需要的证书，选择‘所有任务’，‘导出’，导出证书，如图2.6所示。

​      ![7d8c4f96409d7a1a053dff7c62202895c8a04933](https://yqfile.alicdn.com/7d8c4f96409d7a1a053dff7c62202895c8a04933.jpeg)

​                            图2.6导出证书，第二步

​     7.进入证书导出时，基本都是选择‘默认’直接下一步，下一步就可以，如图2.7所示。

​      ![0107ceeca2a1176fc14c78f8c215cbaec3ad01cf](https://yqfile.alicdn.com/0107ceeca2a1176fc14c78f8c215cbaec3ad01cf.jpeg)

​                            图2.7导出证书，第三步

​     8.如果证书列表里只有一个证书，可能缺少计算机证书，可以根据步骤8，9申请新证书，再参照1-7导出证书，如图2.8所示。

​      ![b0f5be54e2ef87f1ab43489214ae9b072f2f25ea](https://yqfile.alicdn.com/b0f5be54e2ef87f1ab43489214ae9b072f2f25ea.jpeg)

​                            图2.8申请新证书

​     9.申请新证书要选择证书类型为‘计算机’，其他选择默认，如图2.9所示。

​      ![b6188a92c2ec06d5bea17c2ccd6a419425225ff4](https://yqfile.alicdn.com/b6188a92c2ec06d5bea17c2ccd6a419425225ff4.jpeg)

​                            图2.9申请新证书-要点