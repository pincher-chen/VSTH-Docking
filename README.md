# VSTH-Docking

development version: http://172.16.23.111:8008/Myjob.html      
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
     wget https://files.rcsb.org/download/6czj.pdb
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
- 3. 第三步有点多余，刚提交的作业不应该更新。目前网页刷新是遍历每个作业，检查是否是running状态，是的话则执行跟新。需要重新设置策略，刷新作业列表的时候，只刷新每个作业条目，而不是整个网页，这样就不会出现等待刷新完每个作业后才出现网页。
