1.
```sql
select
	m.id,
	m.title,
	m.mission_body,
	s.name,
	mm.status,
	m.reward
from 
	member_mission mm
left join
	mission m ON mm.mission_id = m.id
left join
	store s ON s.id = m.store_id
where
	mm.member_id = ? 
order by
  mm.created_at DESC;
limit 5
offset (?-1)*5
```

2.
```sql
insert into
    review(store_id, member_id, star_rating, review_body)
values
    (’?’, ‘?’, ‘?’, ‘?’)
```

3.
```sql
select
	s.name,
	fc.name,
	m.mission_body,
	m.reward,
	DATEDIFF(mm.due_date, current_date) as days_remaining
from 
	mission m
left join
	member_mission mm ON mm.mission_id = m.id
left join
	store s ON s.id = m.store_id
left join
	food_category fc ON fc.id = s.category_id
where
	s.address = ?
	AND mm.mission_id IS NULL
order by 
	m.created_at DESC
limit 5
offset (?-1)*5
```

4.
```sql
select
	m.id,
	m.name,
	m.email,
	m.point,
	case
		when phone_verified = ‘verified’ then phone_num
		else ‘미인증’
	end as phone_status
from
	member m
where m.id = ?
```