# VSTH-Docking

development version: http://172.16.23.111:8008/Myjob.html   \\   
production version: http://172.16.23.111:8009/Myjob.html   \\ 

## To Do List.
### for front-web

- 1.Add "uuid" and "upload API" for structure-based VS. \\ 
1)Now everytime when the web is loaded, it will create a new uuid. \\ 
2)When clicking the button of "Next", it will post the data of  "uuid/target/file". \\ 
3)And now it need a API for the data posting. \\ 
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
