# VSTH-Docking

debug version: http://172.16.23.111:8008/Myjob.html      
production version: http://172.16.23.111:8009/Myjob.html    

The original VSTH web-server has included the workflow of molecule docking method. However, this workflow is preliminary and needs to be improved. The following points need to be completed.
- 1. Allow users upload protein. The proteins in PDBbind refined-set are supportted in current version, which limits the versatility of VSTH web-server. It needs to check the format of protein files, the preprocess like:
     + assign bond orders
     + add/remove origin hydrogens
     + create zero-order bonds to metals
     + create disulfide bonds
     + delete waters
     + fill in missing side chains
     + .....
  
  Besides, It is better to get pdb form PDB web site, the method of obtaining protein can be implemented as follows:
     ```
     wget https://files.rcsb.org/download/id_code.pdb
     ```
- 2. Add the step of "setting target box size". Three types of size are supportted.
     + recommand by vsth, the default target box size has prepared in database. But, only few proteins are supportted.
     + calculated by predicted softares, like fpocket, sitemap, etc. It will be better that the target sites can be viewed on web.
     + user defined.
- 3. Add more docking softwares. In current version, only Autodock Vina is supportted. Some softares are prepared to add into VSTH in order:
     + schrodinger. Here, only glide module is considered. We will search more info. about how to submit glide job by command.
     + LeDock. This will by easy, for ledock has the same IO files as Vina.
     + AutoDock. 
     + GalaxyDock.
     + iDock.
     + UCSF Dock. I am not sure why the target site do not support.
     + rDock.
     + iGEMDOCK.

## To Do List.
### for front-web

- 1.Add "uuid" and "upload API" for structure-based VS.    
1)Now everytime when the web is loaded, it will create a new uuid.    
2)When clicking the button of "Next", it will post the data of  "uuid/target/file".    
3)And now it need a API for the data posting.    
- 2.For step 2, it needs to post uuid/pdb to sever's backend.


### for back-end

- 1.Tt is too slow when submit a job. Reason for:
```
&{/usr/bin/scp [scp -i /home/chenp/bin/nscc-gz_pinchen.id /data/chenp/code/src/vsth/data/HPVS/ligands/CHEMBL1934618.sdf nscc-gz_pinchen@10.127.48.1:/HOME/nscc-gz_pinchen/WORKSPACE/HHVSF/data/ligands/CHEMBL1934618.sdf] []  <nil> <nil> <nil> [] <nil> <nil> <nil> <nil> <nil> false [] [] [] [] <nil> <nil>}

1
&{/usr/bin/ssh [ssh -i /home/chenp/bin/nscc-gz_pinchen.id nscc-gz_pinchen@10.127.48.1 sh /HOME/nscc-gz_pinchen/WORKSPACE/HHVSF/run_wega_job_slurm.sh 67392bdf_b9b0_4ff9_a330_ec024325da70 CHEMBL1934618.sdf SPECS 4] []  <nil> <nil> <nil> [] <nil> <nil> <nil> <nil> <nil> false [] [] [] [] <nil> <nil>}
data insert successfully.
Submitted batch job 7394395
MongoDB shell version: 3.2.9
connecting to: 172.16.23.111:10000/vs
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
bye
....
&{/usr/bin/ssh [ssh -i /home/chenp/bin/nscc-gz_pinchen.id nscc-gz_pinchen@10.127.48.1 sh /HOME/nscc-gz_pinchen/WORKSPACE/HHVSF/mongo/update_mongo/update_wega_mongo_slurm.sh 67392bdf_b9b0_4ff9_a330_ec024325da70 SPECS] []  <nil> <nil> <nil> [] <nil> <nil> <nil> <nil> <nil> false [] [] [] [] <nil> <nil>}
```

There three steps when a job is finished:
- 1.Scp data from back-end to HPC cluster.
- 2.Submit job by remote ssh command.
- 3.Update running job when redirect to myJob.html.

Method:
分解每一步，让用户知道时间花在什么地方
- 1. 第一步给出上传提示-“正在上传文件”
- 2. 第二步给出提交提示-“正在提交作业”
- 3. 第三步有点多余，刚提交的作业不应该更新。目前网页刷新是遍历每个作业，检查是否是running状态，是的话则执行跟新。需要重新设置策略，局部刷新，即刷新作业列表的时候，只刷新每个作业条目，而不是整个网页，这样就不会出现等待刷新完每个作业后才出现网页。


需要的API：
- 1.step1如果选择Target selection，点击next按钮会向后台传输Json类型数据，例如：
“{"Type":2,"Target":"1B56","jobId":"aef87907_2090_0a46_ed21_c9c90effdb50"}”；
如果选择Template upload，点击next按钮会向后台传输Formdata类型数据，包含jobid和用户上传的file文件，例如：{"Type":2,"Target":"","jobId":"1e6cded9_c008_1782_722f_5479ce907029"}和File(143694)；
需要一个后台API用来接收step1点击next按钮后传输的数据。

- 2.根据step1向后台传输的PDB的相关信息，在step2当中通过相应的API向页面返回对应的坐标数据：
（1）如果选择recommend，点击load按钮后，需要一个API向前端页面返回Size_(x ,y ,z)、Center_(x ,y ,z) 和Num_modes，以Json的格式返回，例如：{"Size_(x ,y ,z)":"10,20,30","Center_(x ,y ,z)":"10,20,30","Num_modes":"10"}；
（2）如果选择calculated，点击绿色字体Calculate后，需要一个API向前端页面返回一系列的Size_(x ,y ,z)、Center_(x ,y ,z)和Scores显示在Target-size下拉菜单当中以供用户选择，以Json格式返回，例如：
{"pockets":[{"score":10,"size_x":10,"size_y":20,"size_z":30,"center_x":10,"center_y":20,"center_z":30},{"score":20,"size_x":10,"size_y":20,"size_z":30,"center_x":10,"center_y":20,"center_z":30}]}
（3）如果选择user-defined，则用户自己填写坐标信息不需要API返回数据。

- 3.当用户在step1~4都选择填写好信息之后，点击submit按钮会将所有信息提交到后台，此时需要一个API来接收所有的数据并处理，所有数据都是Json类型或者Formdata类型的数据。

严鑫建议：

在上述方案的基础上，做用户登录状态的修改，用户的登录状态决定了用户的行为或权限：
1，在现有的版本基础上修改，参考IP地址 http://114.67.37.143:9081/
模式是分为用户登录与未登录情况：
未登录情况：用户为匿名用户，用户可进行对接作业提交，但不能上传自己的靶标PDB文件，或筛选库SDF文件
登录情况：用户可进行对接作业提交，可上传自己的靶标PDB文件，或筛选库SDF文件

对于用户登录情况，业务流程为：
1，用户上传自己的筛选库SDF文件，当前版本在左侧导航栏中已经有my library记录（用户未登录点击my library会提示不可用，需要登录后才能使用）
2，用户上传自己的靶标PDB文件，需要再当前版本中加入 my targets，用于用户上传自己的靶标PDB文件，并存储在服务器文件系统，文件的命令方式为，统一的存储路径 + 用户ID命名的文件夹 + 靶标名称命名的PDB文件，靶标名称由用户上传时设定（不能重复，重复有提示）
3，用户在提交对接作业时，可使用系统提供的靶标库或筛选库，也可以选择自己的靶标文件或筛选库文件，类似当前的系统

对接作业的提交过程：
1，用户选择靶标，可选择系统靶标库、用户自己的靶标文件
   选择系统提供的靶标时，系统靶标的作用位点坐标信息已经计算完毕（不同用户不用重复计算），展示给用户，供选择
   选择用户自己的靶标文件，用户靶标的作用位点信息在用户上传靶标文件时就应该同时计算完毕，并保存信息，这里也是展示给用户选择，不用该用户每次选择某个靶标时重复计算靶标位点信息

2，用户选择需要筛选的数据库，可选择系统提供的筛选库，也可选择用户自己的筛选库

3，选择其他信息（计算节点数量等），提交作业，一个作业提交后会产生唯一ID（jobID）


