# Домашнее задание к занятию «SQL. Часть 2» - Левитский Даниил

Все запросы выполнены в MySQL 8.4 на учебной базе Sakila через DBeaver.

---

### Задание 1

Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:
- фамилия и имя сотрудника из этого магазина;
- город нахождения магазина;
- количество пользователей, закреплённых в этом магазине.

**Решение**

1. Клиенты считаются по `customer.store_id`, группировка по магазину, фильтр по количеству через `HAVING`.
2. Сотрудник магазина берётся из `staff` по `store_id`.
3. Город магазина: `store` → `address` → `city`.
4. Клиенты считаются через `COUNT(DISTINCT ...)`, чтобы при нескольких сотрудниках в магазине число клиентов не задваивалось из-за JOIN.

```sql
SELECT CONCAT(s.last_name, ' ', s.first_name) AS employee,
       c.city,
       COUNT(DISTINCT cu.customer_id) AS customers_count
FROM store st
JOIN staff s     ON s.store_id   = st.store_id
JOIN address a   ON a.address_id = st.address_id
JOIN city c      ON c.city_id    = a.city_id
JOIN customer cu ON cu.store_id  = st.store_id
GROUP BY st.store_id, s.staff_id, s.last_name, s.first_name, c.city
HAVING COUNT(DISTINCT cu.customer_id) > 300;
```

Результат: магазин в городе Lethbridge, сотрудник Hillyer Mike, 326 покупателей.

![Задание 1](https://github.com/bebraclown/12-04-hw/blob/main/img/task1.png)

---

### Задание 2

Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

**Решение**

Средняя продолжительность считается подзапросом, затем выбираются фильмы длиннее этого значения.

```sql
SELECT COUNT(*) AS films_count
FROM film
WHERE length > (SELECT AVG(length) FROM film);
```

Результат: 489 фильмов (средняя продолжительность — 115.27 мин).

![Задание 2](https://github.com/bebraclown/12-04-hw/blob/main/img/task2.png)

---

### Задание 3

Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

**Решение**

1. Дата платежа приводится к виду «год-месяц» через `DATE_FORMAT`, чтобы не смешивать одинаковые месяцы разных лет.
2. Платежи группируются по месяцу, считается сумма `amount` и количество аренд `rental_id`.
3. Сортировка по сумме по убыванию, `LIMIT 1` — месяц с максимальной суммой.

```sql
SELECT DATE_FORMAT(payment_date, '%Y-%m') AS month,
       SUM(amount) AS total_amount,
       COUNT(rental_id) AS rentals_count
FROM payment
GROUP BY month
ORDER BY total_amount DESC
LIMIT 1;
```

Результат: июль 2005 года — сумма платежей 28 368.91, количество аренд 6709.

![Задание 3](https://github.com/bebraclown/12-04-hw/blob/main/img/task3.png)
