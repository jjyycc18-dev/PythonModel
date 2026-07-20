# 아래 쿼리를 datalake에 저장하는데 space_datamart에 저장하는 코딩을 만들어라
'''
select s.k_dsm_seq, s.index_no
  ,case when t.eqp_id='N' then 'N' else s.eqp_id end eqp_id
  ,case when t.chamber_id='N' then 'N' else s.chamber_id end chamber_id
  ,case when t.ppid='Y' then s.ppid else t.ppid end ppid
  ,case when t.reticle_id='Y' then s.reticle_id else t.reticle_id end reticle_id
  ,case when t.pre_eqp_id='N' then 'N' else s.pre_eqp_id end pre_eqp_id
  ,s.lot_cnt
  ,s.time_gap_hrs
  ,s.create_tmstp
  ,s.update_tmstp
  ,s.pre_measure_ppid
  ,s.sched_measure_ppid
  ,s.init_cycle_yn
  ,t.measure_ppid_rottn_yn
  ,s.cur_init_cycle_cnt
from ivm_k_ds_ovl_status s
join( 
  select c.k_dsm_seq, 0 index_no, c.eqp_id, c.ppid, c.reticle_id, c.pre_eqp_id, c.measure_ppid_rottn_yn, c.chamber_id
  from ivm_k_ds_ovl_config c
  where c.k_dsm_seq in (select k_dsm_seq from ivm_k_ds_ovl_config) 
  union
  select c.k_dsm_seq, dt.index_no, dt.eqp_id, dt.ppid, dt.reticle_id, dt.pre_eqp_id, c.measure_ppid_rottn_tn, dt.chamber_id
  from ivm_k_ds_ovl_config c
  join ivm_k_ds_ovl_config_dt dt on c.k_dsm_seq = dt.k_dsm_seq
  where c.k_dsm_seq in (select k_dsm_seq from ivm_k_ds_ovl_config) 
) t on s.k_dsm_seq = t.k_dsm_seq and s.index_no = t.index_no
where s.k_dsm_seq in (select k_dsm_seq from ivm_k_ds_ovl_config);
'''

여기에 아래 코딩을 참조하여 위 쿼리를 코딩을 해라
'''
from common import fetch_data, constants
from config import config
from dao import vm_dao
from net.space_request_client import HttpRequestClient
import logging
import pandas as pd
logger = logging.getLogger(__name__)

def bigdataquery_decorator(func):
    def func_wrapper(*args, **kwargs):
        logger.info("bigdataquery_dao.{0} {1}".format(func.__name__, "called."))
        result = None
        try:
            result = func(*args, **kwargs)
        except Exception as e:
            logger.exception("bigdataquery_dao.{0} {1} : {2}".format(func.__name__, "unexpected exception occurred...", e))
        finally:
            pass
        logger.info("bigdataquery_dao.{0} {1}".format(func.__name__, "completed."))
        return result
    return func_wrapper

@bigdataquery_decorator
def get_eqp_hw_p_idle_history(target_line, eqp_id, start_date, end_date, lot_id):
    sql_query =(f"""
                     select lotid, if_lot_id, equipmentid, moduleid, workgroup, state, recipename, stepname, starttime_rev, endtime_rev, materialid, if_step_seq
                       from tab.m_fab_process
                      where equipmetid = '{eqp_id}'
                        and dateonly >= '{start_date} 00:00:00'
                        and dateonly <= '{end_date} 00:00:00'
                        and targetline = '{target_line}'
                        and (if_lot_id = '{lot_id}' or lotid = '{lot_id}')    
                  """)
    param_dict = {"query": sql_query}
    rc = HttpRequestClient(config.space_db_if_service['bigdataquery_getdata_sql'], param_dict, 60 * 10 * 2)
    df =  pd.read_json(rc.get_result(), dtype={'starttime_rev': 'datetime64', 'endtime_rev': 'datetime64' })

    return df
'''    
