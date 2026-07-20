# 아래 쿼리를 bigdataquery_dao에 저장하는 코딩을 만들어라

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
