# `datalad-status`
## Logic flow of method `datalad_status()` of class BABS

* create `job_status.csv` if it does not exist yet
* get 'keywords_alert' configs
* get username (to be used by `qacct` job accounting)
* get list of branches in output RIA with `git branch -a` (NOTE: this is quick, even for tons of branches)
* with `job_status.csv` file opened:
    * get original `df_job`
    * make new one `df_job_updated` (copy of original one)
    * request `qstat` for **all** jobs: `df_all_job_status`
    * for each job that has been submitted but not `is_done`:
        * get basic information of this job
        * get last line of `.o` file
        * check if any alert message in the log files (based on 'keywords_alert')
        * if there is a branch of current job in output RIA, the job is done, and update `df_job_updated`
        * if not, the job is pending/running/failed/eqw:
            * if the job is in the queue `df_all_job_status`, i.e., is pending/running/eqw:
                * if `r`:
                    * if `--resubmit-job` for this job & `--reckless`: resubmit
                    * else: update `df_job_updated`
                * if `qw`:
                    * resubmit if 'pending' in `flags_resubmit`, or request specifically: resubmit and update `df_job_updated`
                * if `eqw`: **TODO**
            * else, i.e., not in the queue, so failed:
                * update `df_job_updated`
                * resubmit if 'failed' in `flags_resubmit`, or request specifically: resubmit and update `df_job_updated`
                * if did not resubmit:
                    * if `--job-account` and no alert keywords in logs:
                        * do `qacct`, and update 'job_account' column.

    * for each job that marked as "is_done" in previous round:
        * if `--resubmit-job` for this job & `--reckless`: resubmit
        * else:
            * get last line of `.o` file. Purpose: when marked as 'is_done' (got branch in output RIA), the job hasn't been finished yet, and needs to do `datalad drop` etc before echoing 'SUCCESS'. This is to make sure that we can get 'SUCCESS' for 'last_line_o_file' for 'is_done' jobs.
            * check if any alert message in the log files (based on 'keywords_alert'); Purpose: update it for successful jobs too in case user updates the configs in yaml file

    * for jobs that haven't been submitted yet:
        * if `--resubmit-job` is requested, check if any requested jobs are not submitted yet; if so, throw out a warning

    * save `df_jobs_updated`
    * summarize the job status and report

Summary:
- 'keywords_alert' is detected in all submitted jobs, no matter 'is_done' in previous round or not

## Resubmit based on job's status:
| job status | what to do if resubmit is requested | progress | tested? |
| :-- | :--|:-- | :-- |
| not submitted | warning: `babs-submit` first | added | edge case, not tested yet |
| submitted, qw | resubmit | added | tested with multi-ses data |
| submitted, running | with `--reckless`, resubmit; else, warning, not to resubmit | added | edge case, not tested yet |
| submitted, eqw | resubmit | added | not tested yet, as cannot enter eqw... |
| submitted, failed | resubmit | added | tested with multi-ses data |
| submitted, is_done | with `--reckless`, resubmit; else, warning, not to resubmit | added, one TODO | not tested yet |


# Example `job_status.csv`
## When just initialized:
```
sub_id,ses_id,has_submitted,job_id,job_state_category,job_state_code,duration,is_done,is_failed,log_filename,last_line_o_file,alert_message,job_account
sub-01,ses-A,False,-1,,,,False,,,,,
```
when `print(df)` by python:
```
   sub_id ses_id  has_submitted  job_id  job_state_category  job_state_code  \
0  sub-01  ses-A          False      -1                 NaN             NaN

   duration  is_done  is_failed  log_filename  last_line_o_file  alert_message  job_account
0       NaN    False        NaN           NaN               NaN            NaN          NaN
```
(`0` at the beginning: index of pd.DataFrame)

# Testing
## Create pending, failed, or stalled jobs
Change/Add these in `participant_job.sh`:
- failed: add `-l h_rt=0:0:20` (hard runtime limit is 20 sec)
- pending: increase `-l h_vmem` and `-l s_vmem`; increase `-pe threaded N`
- stalled (`eqw`): see Bergman email 12/20/22

After these changes, `datalad save -m "message"` and `datalad push --to input`

# Terminology

- "<jobname>.o<jobid>": standard output stream of the job
- "<jobname>.e<jobid>": standard error stream of the job