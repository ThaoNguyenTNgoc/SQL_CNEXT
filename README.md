# SQL_CNEXT
SQL CNext daily
-- check sót bonus_app
select f."id" as fin_id, a.ref_user, a.process_date,a.tracking_id, m.map_code, m.release_phase, a.ref_user, s."name" from project_apps a 
join project_status_map m on m."id"= a."process_status_map" and m.release_phase is not null 
left join project_apps_bonuses b on b."app"= a."id" 
left join project_apps_fin f on f."app"= a."id"
left join project_apps_amb amb on amb."app"= a."id" 
join project_schemes s on s."id"= f.project_scheme or s."id"= amb.project_scheme
where s."id"!='c76c739b-5d7b-486e-bfe9-612a8a5b8039'
and a.ref_user !='50738d80-6895-49a2-93d9-f8a8c39f4f24'
and b."id" is null order by a.process_date

-- danh sách user khoá code
select p.code, u.ref_code, bl.created_at, sb.status, sb.note from users u 
join project_role_subscriptions sb on sb."user"= u."id"
join project_bonuses bn on bn."id"= sb.project_bonus
join project_role_blacklist bl on bl."user"=  u."id" and bl.project_bonus= bn."id"
join projects p on p."id"= bn."project"
where p.status='20' ;

-- check time_out
SELECT e.payload,eil.request, response, eil.created_at::time ,e.* from event_invocation_logs eil
join event_log e on eil.event_id = e."id"
where  eil.created_at > '2025-12-10 03:00:00'
and not eil.created_at::time BETWEEN '15:00:00' and '16:00:00'
and delivered = false
ORDER BY eil.created_at DESC;



select u."id",u.ref_code, u.phone, u.money, tbl.sum_thu_nhap, (u.money-tbl.sum_thu_nhap) as lech_money  from users u 
join (
select sum(amount) as sum_thu_nhap, "user" from transactions group by "user") tbl on tbl."user"=u."id"
where u."money" != tbl.sum_thu_nhap;

select * FROM transactions
WHERE  "user"='24c255a4-021e-4982-b29b-520e8f5f0330'

select * from queues where payload::json->>'id'='43407bc5-660b-4836-bbbd-bd381fe1c986'



select a.tracking_id, f.code,s.code, f.disbursed_date, b.bonus from project_apps a 
join project_apps_fin f on f."app"= a."id"
join project_apps_bonuses b on b."app"= a."id"
join project_status s on s."id"= a.process_status
where a.project='22d32fbf-0606-4d45-86f4-418a247026f8'
and f.disbursed_date >='2025-10-31 17:00:00' and f.disbursed_date <'2025-11-30 17:00:00' 
and b.bonus_type='50' order by s.code


--- tax_request_weekly

select t.created_at,u.full_name, u.phone, p.id_number, p.id_number_old, p.date_of_birth, p.place_of_issue_id_number,
p.issue_date_id_number, p.permanent_address, p.address_detail, a1."name"
, a2."name", a3."name", p.id_number_front_image, p.id_number_back_image,p.id_number_selfie_image from users u 
join user_profiles p on p."user"= u."id"
join user_taxes t on t."user"= u."id"
join master_address a1  on a1."id"= p.province 
join master_address a2  on a2."id"= p.district 
join master_address a3  on a3."id"= p.ward 
where 
t.verify_tax_type= '7' and t.tax_status ='14' 
and t.deleted='f' 
order by t.created_at
