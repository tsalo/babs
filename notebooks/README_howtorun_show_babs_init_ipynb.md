## How to generate html file from this ipynb:
NOTE: if on cubic, please run all commands in a terminal! DO NOT USE cubic + vscode!!
1. set up the terminal:
```
source ~/.bashrc   # freesurfer and templateflow env variables
conda activate mydatalad
```
2. manually install BABS:
```
cd ~/babs
pip install -e .
```
3. then run:
```
jupyter nbconvert --execute --to html notebooks/<your_ipynb_filename>.ipynb
```

## After `babs-init` is run:
- save the printed messages to `analysis/code`
    - then: `datalad save`, `datalad push --to input`, `datalad push --to output`

## After all jobs are successfully run:
- first, go to `output_ria/xxx/xxx-xx-xxx`, `git branch -a`. 
    - Make sure all branches are there, 
    - and there is no extra/repeated branches of the same sub/session
If the list of branches looks good to you:
- merge: `[analysis] $ bash code/merge_outputs.sh`
- clone out output_ria to see

## How to interactively test out the bootstrapped scripts:
- open `participant_job.sh` and `submit_jobs.sh`
- enter the arguments value into terminal based on `submit_job.sh`
- `$ where_to_run="/cbica/comp_space/$(basename $HOME)"   # using comp_space `   

## After the job is gone from the list, check:
- output_ria/xxx/xxx-xxx-xxx-xxx: $ git branch -a  # to see if the job is success
- `analysis/logs` for logs
- cd "/cbica/comp_space/$(basename $HOME)"   # comp_space to find out that job
- qaccj -j <jobID> for diagnosis

## How to remove a BABS project:
```
cd <project_root>/analysis
datalad remove -d inputs/data/<input_ds_name>
git annex dead here
datalad push --to input
datalad push --to output
cd ../..   # outside of <project_root>
rm -rf <project_root>
``` 

## How to remove temporary workspace (from `comp_space`):
```
# cd to `comp_space`
cd <job-name>/ds
datalad drop -r --reckless availability --reckless modification -d inputs/data/<input_ds_name>
datalad drop -r . --reckless availability --reckless modification
git annex dead here
pwd    # and please copy the <job-name>
cd ../..   # outside of <job-name>
rm -rf <job-name>
```