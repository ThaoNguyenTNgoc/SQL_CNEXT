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



--- check postback ekyc
select * from users u 
join postback_log l on l."user"= u."id"
join user_device ud on ud."user"= u."id"
join device d on d."id"= ud.device
where l.status='PASS' and u.identity_status='4' 
and u."id" != 'cfcd8b1b-64fe-47bb-b62e-a0a590ab317f'
and ud.deleted='f'
-- group by  u."id", u.ref_code,l.response_at
order by l.response_at desc

--- check bonus phát sinh trong tuần
select count(a.tracking_id) as count, a.project, b.bonus_total from project_apps_bonuses b
join transactions t on t."id"= b."transaction"
join project_apps a on a."id"= b.app
where b.created_at>='2025-12-14 17:00:00' and b.created_at<'2025-12-21 17:00:00'
and b.bonus_type='50'
group by a.project, b.bonus_total


-- check đơn truy thu 
select u.phone, u.ref_code, a.tracking_id, a.cnext_id,a.process_note, b.bonus, t.description, t1.description from project_apps a 
join users u on u."id" = a.ref_user 
join project_apps_bonuses b on b."app"= a."id" 
join transactions t on t."id"= b."transaction" 
left join transactions t1 on t1."ref_transaction"= t."id" 
where a.tracking_id in ('25349109',
'26030229',
'26164406',
'26370024');

select u."id", u.phone, u.ref_code,u.money, a.tracking_id, a.cnext_id from project_apps a 
join users u on u."id" = a.ref_user 
-- join project_apps_bonuses b on b."app"= a."id" 
-- join transactions t on t."id"= b."transaction" 
-- left join transactions t1 on t1."ref_transaction"= t."id" 
where a.tracking_id in ('25349109',
'26030229',
'26164406',
'26370024')
order by a.tracking_id

---
select w.code, u.ref_code, u."id" from users u 
join withdraw w on w."user"= u."id"
where u.money<0 and w.status='50'

---
select * from queues where payload::json->>'id'='892878e6-6637-481b-9333-950fa38d1a6d'
select * from user_taxes where "user" in (select "id" from users where phone='0399329406');
select * from user_taxes where "user" in (select "id" from users where phone='0978146472');
select * from user_taxes where "user" in (select "id" from users where phone='0362494715');


select count(*) as count, nickname from users group by nickname
having count(*)>1;

select * from project_apps where tracking_id in ('25476711')

-- test task crontab quét project_inspections
SELECT a.tracking_id, b.app,s."name", i.* FROM "project_inspections" i 
join project_apps a on a."id"= i."project_app"
join master_status  s on s."id"= i.status
left join 
(select app from project_apps_bonuses group by app) b on b."app"= a."id"
order by i.created_at 

