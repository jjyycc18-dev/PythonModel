oco dsm filter 기능개발 플로우
]
1. oco dsm setrting 조건 [filter use yn] & [auto filter use yn] 이 y 일 경우 [auto filter dr](design rule) 입력이 가능해지며 auto filter가 동작한다.

2. s3 drive에 있는 incoming_results.csv파일을 읽는다 
  ( aria_overlay_results\D1b\20260601\wrapup\incoming_results.csv ) 예시 : 가장 최근일자의 폴드 검색 "20260601"이 가장최근이라고 본경우

예시
'''
  import pprint
  res = client.list_objects_v2(Bucket-"dsm_oco_auto_filtering")
  pprint.pprint(res)
'''

3. csv 파일 내 기준 정보 및 filter 정보 수집

'''
-csv 파일 구조
-칼럼명정보 (1번째줄은 main정보, 2번째줄은 filter정보 라 하자)
  process_id, current_step, current_eqpid, current_chamberid, current_ppid, current_reticleid, prev_step, prev_step_eqpid, slot_step, trigger_type, trigger_step, DSM_step, 
  operation, filter1_stepseq, filter2_ppid, filter3_eqpid, filter4_chamberid, filter5, confirmed by, comment
-data 정보
  KGVC, VC075030, Y, N , Y, N VC046030, Y , VC075040, TKIN, VC077090, VC077251 , add , VC077090, ALL, EAPGF2, B, ALL, nan, comment111
  KGVC, VC075030, Y, N , Y, N VC046030, Y , VC075040, TKIN, VC077090, VC077251 , add , VC077090, ALL, EAPGF2, B, ALL, nan, comment111
'''

4. csv파일내 [operation] 컬럼이 'add'이면 filter정보 등록(is auto : Y 포함), 'delete'이면 기존 등록된 filter정보 삭제
   - 등록 및 삭제시 csv파일에서 파싱한 oco dsm main정보 사용
   - filter정보 등록/삭제시 매칭조건에서 trigger_step은 제외
   - 만약 전달받은 filter1의 stepseq 정보가 기존에 main정보에 등록된 trigger_step보다 뒤일 경우 filter1의 정보로  trigger_step을 치환( 3번째 부터 6개의 숫자로 계산)
      (예: trigger step : VH077040, filter1 : VH077090인 경우 VH077090으로  trigger step을 변경한다

5. 필터정보 테이블 ( ivm_k_ds_ovl_filter )
'''
no, 컬럼명, 타입, 칼럼설명
------------------
01, k_dsm_seq, number, 설정번호 key값
02, k_dsm_filter_seq,  number, filter설정번호 key값
03, filter_type_code, varchar2, 필드종류
04, cond_1st_type, varchar2, 필터조건
05, cond_2nd_type, varchar2, 필터조건
06, cond_3rd_type, varchar2, 필터조건
07, cond_4th_type, varchar2, 필터조건
08, cond_5th_type, varchar2, 필터조건
09, filter_order, number, filter순서
10, creatr_seq, number,
11, create_tmstp, timestamp, 
12, updater_seq, number, 
13, update_tmstp, timestamp, 
14, n_lot_dsm_yn, varchar2, 
15, dsm_lot_cnt,number ,
16, dsm_slot_list, varchar2, 
17, dsm_reset_tmstp, timestamp, 
18, filter_comment, varchar2, filter설정에대한 커멘트

'''

































