# Лабораторная работа 1

## Задание: Система записи пациентов к врачам

Имеются **Пациенты** (ФИО, дата рождения, контактный телефон) и **Врачи** (ФИО, специальность, категория). Пациент может записаться на прием к нескольким врачам, причем к каждому врачу — на определенную дату и время.

### Выходные документы (запросы)
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
Преобразование ER-диаграммы в логическую модель
Сущности и атрибуты:

PATIENTS (Пациенты):

id: INTEGER (Первичный ключ)

full_name: VARCHAR(40) (ФИО)

date_of_birth: DATE (Дата рождения)

phone_number: VARCHAR(50) (Контактный телефон)

DOCTORS (Врачи):

id: INTEGER (Первичный ключ)

full_name: VARCHAR(40) (ФИО)

speciality: VARCHAR(50) (Специальность)

category: VARCHAR(50) (Категория)

VISITS (Записи на прием):

id: INTEGER (Первичный ключ)

patient_id: INTEGER (Внешний ключ к PATIENTS.id)

doctor_id: INTEGER (Внешний ключ к DOCTORS.id)

date: DATE (Дата приема)

time_visit: TIME (Время приема)

Связи:

Один пациент может иметь много записей (1:M)

Один врач может иметь много записей (1:M)
