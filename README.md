# Лабораторная работа 1. Проектирование структуры БД

## Выбор задачи

**Задание:** Система записи пациентов к врачам

Имеются **Пациенты** (ФИО, дата рождения, контактный телефон) и **Врачи** (ФИО, специальность, категория). Пациент может записаться на прием к нескольким врачам, причем к каждому врачу — на определенную дату и время.

**Выходные документы (запросы):**
1. Выдать список пациентов, записанных к указанному врачу, упорядоченный по ФИО.
2. Выдать список пациентов и количество запланированных у них приемов, упорядоченный по ФИО.

## ER-диаграмма

```mermaid
erDiagram
    PATIENTS {
        int id PK "Идентификатор"
        varchar full_name "ФИО"
        date date_of_birth "Дата рождения"
        varchar phone_number "Контактный телефон"
    }
    
    DOCTORS {
        int id PK "Идентификатор"
        varchar full_name "ФИО"
        varchar speciality "Специальность"
        varchar category "Категория"
    }
    
    VISITS {
        int id PK "Идентификатор"
        int patient_id FK "ID пациента"
        int doctor_id FK "ID врача"
        date date "Дата приема"
        time time_visit "Время приема"
    }

    PATIENTS ||--o{ VISITS : "записывается"
    DOCTORS ||--o{ VISITS : "принимает"
```

## Логическая модель

**PATIENTS (Пациенты):**
- id: INTEGER (Первичный ключ)
- full_name: VARCHAR(40) (ФИО)
- date_of_birth: DATE (Дата рождения)
- phone_number: VARCHAR(50) (Контактный телефон)

**DOCTORS (Врачи):**
- id: INTEGER (Первичный ключ)
- full_name: VARCHAR(40) (ФИО)
- speciality: VARCHAR(50) (Специальность)
- category: VARCHAR(50) (Категория)

**VISITS (Записи на прием):**
- id: INTEGER (Первичный ключ)
- patient_id: INTEGER (Внешний ключ к PATIENTS.id)
- doctor_id: INTEGER (Внешний ключ к DOCTORS.id)
- date: DATE (Дата приема)
- time_visit: TIME (Время приема)

**Связи:**
- Один пациент может иметь много записей (1:M)
- Один врач может иметь много записей (1:M)

---

# Лабораторная работа 2. Инсталляция БД на сервере

## Физическая модель для PostgreSQL

```sql
CREATE TABLE IF NOT EXISTS belousov2262.doctors(
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(40),
    speciality VARCHAR(50),
    category VARCHAR(50)
);

CREATE TABLE IF NOT EXISTS belousov2262.patients(
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(40),
    date_of_birth DATE,
    phone_number VARCHAR(50)
);

CREATE TABLE IF NOT EXISTS belousov2262.visits(
    id SERIAL PRIMARY KEY,
    patient_id INT REFERENCES belousov2262.patients(id),
    doctor_id INT REFERENCES belousov2262.doctors(id),
    date DATE,
    time_visit TIME
);
```

## Заполнение таблиц данными

```sql
INSERT INTO belousov2262.doctors (full_name, speciality, category) VALUES
('Иванов Петр Сергеевич', 'Терапевт', 'Высшая'),
('Сидорова Анна Владимировна', 'Кардиолог', 'Первая'),
('Петров Михаил Андреевич', 'Невролог', 'Высшая'),
('Козлова Елена Ивановна', 'Эндокринолог', 'Вторая');

INSERT INTO belousov2262.patients (full_name, date_of_birth, phone_number) VALUES
('Смирнов Алексей Иванович', '1985-03-15', '+7 (912) 345-67-89'),
('Волкова Татьяна Петровна', '1992-07-22', '+7 (923) 456-78-90'),
('Кузнецов Андрей Сергеевич', '1978-11-30', '+7 (934) 567-89-01'),
('Попова Екатерина Викторовна', '2001-05-18', '+7 (945) 678-90-12');

INSERT INTO belousov2262.visits (patient_id, doctor_id, date, time_visit) VALUES
(1, 1, '2024-01-15', '09:30:00'),
(2, 2, '2024-01-16', '10:15:00'),
(3, 3, '2024-01-17', '11:00:00'),
(1, 2, '2024-01-18', '14:20:00'),
(2, 1, '2024-01-19', '15:45:00'),
(3, 1, '2024-01-20', '08:30:00'),
(4, 4, '2024-01-21', '12:10:00');
```

## Проверка нормальных форм

**1NF (Первая нормальная форма):**
- ✓ Все таблицы имеют первичные ключи
- ✓ Все атрибуты атомарны (не содержат составных значений)
- ✓ Нет повторяющихся групп данных

**2NF (Вторая нормальная форма):**
- ✓ Все таблицы находятся в 1NF
- ✓ Все неключевые атрибуты полностью зависят от первичного ключа
- ✓ В таблице `visits` все атрибуты зависят от полного ключа `id`

**3NF (Третья нормальная форма):**
- ✓ Все таблицы находятся в 2NF
- ✓ Отсутствуют транзитивные зависимости
- ✓ Например, в таблице `doctors` нет зависимостей между специальностью и категорией

**4NF (Четвертая нормальная форма):**
- ✓ Отсутствуют многозначные зависимости
- ✓ В таблице `visits` каждый атрибут независим

**5NF (Пятая нормальная форма):**
- ✓ Все таблицы находятся в 4NF
- ✓ Отсутствуют зависимости соединения без потерь
- ✓ Структура данных позволяет естественные соединения без избыточности

**Вывод:** Все таблицы соответствуют требованиям до 5NF включительно.

## Создание дампа базы данных

```bash
pg_dump -n belousov2262 --inserts -U postgres -d database_name > belousov2262_dump.sql
```

**Выполнение содержательных SELECT-запросов**

**Запрос 1:**
```sql
SELECT p.id, p.full_name 
FROM belousov2262.patients AS p
JOIN belousov2262.visits AS v ON v.patient_id = p.id
JOIN belousov2262.doctors AS d ON d.id = v.doctor_id
WHERE d.id = 1
ORDER BY p.full_name;
```

**Запрос 2:**
```sql
SELECT p.id, p.full_name, COUNT(v.id) as visits_count
FROM belousov2262.patients AS p
LEFT JOIN belousov2262.visits AS v ON v.patient_id = p.id
GROUP BY p.id, p.full_name
ORDER BY p.full_name;
```
**Результат Запроса 1:**

<img width="330" height="196" alt="image" src="https://github.com/user-attachments/assets/968ca56d-12d3-4c9e-836e-197dd5a2f47f" />


**Результат Запроса 2:**

<img width="437" height="393" alt="image" src="https://github.com/user-attachments/assets/080e5a90-0fcf-4a5d-9d45-0d46b3c9b56c" />

# Лабораторная работа 3. Представления и процедуры

## 1. Создание представлений для выходных документов

**Представление 1:** Пациенты по врачам
```sql
CREATE OR REPLACE VIEW belousov2262.patients_by_doctor AS
SELECT 
    p.id,
    p.full_name,
    p.date_of_birth,
    p.phone_number,
    d.id as doctor_id,
    d.full_name as doctor_name,
    d.speciality,
    v.date,
    v.time_visit
FROM belousov2262.patients p
JOIN belousov2262.visits v ON p.id = v.patient_id
JOIN belousov2262.doctors d ON v.doctor_id = d.id;
```

**Использование:**
```sql
SELECT id, full_name, date, time_visit
FROM belousov2262.patients_by_doctor
WHERE doctor_id = 1
ORDER BY full_name;
```
**Результат использования:**

<img width="547" height="189" alt="image" src="https://github.com/user-attachments/assets/fe219812-0514-4972-aafe-51fac1ef55d7" />


**Представление 2:** Статистика посещений пациентов
```sql
CREATE OR REPLACE VIEW belousov2262.patient_visit_stats AS
SELECT 
    p.id,
    p.full_name,
    p.date_of_birth,
    p.phone_number,
    (SELECT COUNT(*) 
     FROM belousov2262.visits v 
     WHERE v.patient_id = p.id) as total_visits
FROM belousov2262.patients p;
```

**Использование:**
```sql
SELECT id, full_name, total_visits
FROM belousov2262.patient_visit_stats
ORDER BY full_name;
```
**Результат использования:**

<img width="409" height="392" alt="image" src="https://github.com/user-attachments/assets/bf601142-a3a5-4a2a-909d-dccdd3cb24d9" />



## 2. Разработка хранимых процедур с параметрами

**Процедура 1:** Добавление нового врача
```sql
CREATE OR REPLACE PROCEDURE belousov2262.add_doctor(
    p_full_name VARCHAR,
    p_speciality VARCHAR,
    p_category VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO belousov2262.doctors (full_name, speciality, category)
    VALUES (p_full_name, p_speciality, p_category);
END;
$$;
```

**Процедура 2:** Запись пациента на прием
```sql
CREATE OR REPLACE PROCEDURE belousov2262.book_appointment(
    p_patient_id INT,
    p_doctor_id INT,
    p_date DATE,
    p_time TIME
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO belousov2262.visits (patient_id, doctor_id, date, time_visit)
    VALUES (p_patient_id, p_doctor_id, p_date, p_time);
END;
$$;
```


## 3. Представление сложных запросов при помощи представления

**Сложное представление:** Полная статистика по врачам
```sql
CREATE OR REPLACE VIEW belousov2262.doctor_full_statistics AS
SELECT 
    d.id,
    d.full_name,
    d.speciality,
    d.category,
    (SELECT COUNT(*) 
     FROM belousov2262.visits v 
     WHERE v.doctor_id = d.id) as total_appointments
FROM belousov2262.doctors d;
```

**Использование сложного представления:**
```sql
SELECT full_name, speciality, total_appointments
FROM belousov2262.doctor_full_statistics
ORDER BY total_appointments DESC;
```
**Результат использования:**

<img width="539" height="338" alt="image" src="https://github.com/user-attachments/assets/5aa66752-afbe-413b-96be-bd1ffd66a9d4" />


## Проверка работы

**Проверка представлений:**
```sql
SELECT * FROM belousov2262.patients_by_doctor WHERE doctor_id = 1 LIMIT 3;
SELECT * FROM belousov2262.patient_visit_stats LIMIT 3;
```

**Проверка процедур:**
```sql
CALL belousov2262.add_doctor('Новиков Иван Сергеевич', 'Хирург', 'Первая');
CALL belousov2262.book_appointment(1, 2, '2024-02-01', '14:30:00');
CALL belousov2262.get_doctor_schedule(1, '2024-01-19');
```

## Результаты выполнения

**Результат использования представления patients_by_doctor:**
<img width="1189" height="121" alt="image" src="https://github.com/user-attachments/assets/b2d18c1e-5c9a-4641-8639-49385d2bf498" />



**Результат использования представления patient_visit_stats:**
<img width="802" height="116" alt="image" src="https://github.com/user-attachments/assets/ae0f39d0-6a87-4471-8d69-9f4ea4313238" />

