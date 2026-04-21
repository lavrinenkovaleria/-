## 2. Описание проекта

**Название проекта:** «Учебный проектный офис» (Student Project Manager)  

**Назначение:**  
Веб-приложение для студентов, преподавателей и руководителей проектов, позволяющее создавать учебные проекты, распределять задачи, отслеживать дедлайны, сдавать отчеты и оценивать работы.

**Выявленные проблемы:**
- Хаотичное хранение файлов по проектам
- Пропуск дедлайнов из-за отсутствия единого календаря
- Сложность отслеживания вклада каждого студента
- Отсутствие прозрачной системы сдачи и проверки заданий

**Решение:**
- Единое пространство для проектов с ролевой моделью
- Система задач с дедлайнами и уведомлениями
- Чат и комментарии внутри проекта
- Хранение версий файлов
- Журнал оценок и обратная связь

**Целевая аудитория:**

| Роль | Потребности |
|------|--------------|
| Студент | Создавать проекты, сдавать задания, видеть оценки, общаться в команде |
| Руководитель проекта (студент) | Управлять командой, назначать задачи, следить за прогрессом |
| Преподаватель | Создавать шаблоны проектов, проверять работы, выставлять баллы |
| Администратор | Управлять пользователями, группами, курсами |

**Решаемые задачи:**
1. Централизованное управление учебными проектами
2. Контроль соблюдения дедлайнов
3. Прозрачная оценка вклада каждого участника
4. Автоматизация уведомлений о сдаче/проверке работ

---

## 3. Формализация требований

### 3.1 Функциональные требования

**Роль «Студент / Руководитель проекта»**

| ID | Требование | Описание |
|----|-------------|----------|
| F01 | Авторизация | Вход по email/паролю или через корпоративный портал |
| F02 | Управление проектами | Создание, редактирование, архивация проекта |
| F03 | Управление задачами | Создание задач, назначение ответственных, дедлайны, статусы |
| F04 | Загрузка файлов | Прикрепление отчетов, презентаций, кода |
| F05 | Комментарии | Обсуждение задач и файлов внутри проекта |
| F06 | Просмотр оценок | Просмотр баллов и обратной связи |
| F07 | Календарь | Отображение всех дедлайнов |

**Роль «Преподаватель»**

| ID | Требование | Описание |
|----|-------------|----------|
| A01 | Просмотр всех проектов | Доступ к проектам своей учебной группы |
| A02 | Проверка заданий | Оценка, комментарии, статус "Принято/Доработать" |
| A03 | Создание шаблонов | Типовые структуры проектов для студентов |
| A04 | Статистика группы | Успеваемость по проектам, средний балл |

### 3.2 Нефункциональные требования

| ID | Требование | Описание | Критерий |
|----|-------------|----------|----------|
| N01 | Производительность | Загрузка списка проектов | ≤ 2 секунд |
| N02 | Надёжность | Автосохранение комментариев | Каждые 30 секунд |
| N03 | Доступность | Веб + адаптив | Mobile First |
| N04 | Безопасность | Разграничение доступа (RBAC) | Студент не видит чужие проекты |
| N05 | Вместимость | Хранение файлов | До 100 МБ на проект |
| N06 | Удобство | Drag-and-drop файлов | Да |

---

## 4. Технологический стек

| Компонент | Технология | Обоснование |
|-----------|------------|--------------|
| Frontend | React + Vite | Быстрая разработка, компоненты |
| UI Library | Material UI (MUI) | Готовые компоненты для таблиц, форм, календаря |
| State Manager | Redux Toolkit | Проекты, задачи, пользователь |
| Backend | Node.js + Express | REST API, простота |
| Database | PostgreSQL | Надёжность, связи "многие ко многим" |
| File Storage | MinIO / S3 | Хранение файлов проектов |
| ORM | Prisma | Современный, типобезопасный |
| Auth | JWT + bcrypt | Токены, хеширование |
| Hosting | Vercel + Railway | Бесплатные тарифы для учебного проекта |

---

## 5. Модель данных
<img width="846" height="811" alt="image" src="https://github.com/user-attachments/assets/35d677f1-538d-48ff-87f6-2dee8607d269" />


### 5.2 Описание таблиц

**Таблица: Users**

| Поле | Тип | Описание | Ограничения |
|------|-----|----------|--------------|
| id | SERIAL | ID пользователя | PRIMARY KEY |
| email | VARCHAR(100) | Почта | UNIQUE, NOT NULL |
| passwordHash | VARCHAR(255) | Пароль | NOT NULL |
| fullName | VARCHAR(100) | ФИО | NOT NULL |
| role | ENUM | student/teacher/admin | DEFAULT 'student' |
| groupName | VARCHAR(50) | Учебная группа | |

**Таблица: Projects**

| Поле | Тип | Описание | Ограничения |
|------|-----|----------|--------------|
| id | SERIAL | ID проекта | PRIMARY KEY |
| ownerId | INT | Создатель | FOREIGN KEY → Users(id) |
| title | VARCHAR(200) | Название | NOT NULL |
| description | TEXT | Описание | |
| deadline | DATE | Дедлайн | |
| status | ENUM | active/archived | DEFAULT 'active' |

**Таблица: ProjectMembers (M:N)**

| Поле | Тип | Ограничения |
|------|-----|--------------|
| projectId | INT | FOREIGN KEY → Projects(id) |
| userId | INT | FOREIGN KEY → Users(id) |
| roleInProject | ENUM | member/leader |
| PRIMARY KEY (projectId, userId) | | |

**Таблица: Tasks**

| Поле | Тип | Описание |
|------|-----|----------|
| id | SERIAL | PRIMARY KEY |
| projectId | INT | FOREIGN KEY → Projects(id) |
| title | VARCHAR(200) | NOT NULL |
| assigneeId | INT | FOREIGN KEY → Users(id) |
| deadline | DATE | |
| status | ENUM | todo/in_progress/done |
| grade | INT | Оценка от 0 до 100 |

**Таблица: Comments**

| Поле | Тип |
|------|-----|
| id | SERIAL PRIMARY KEY |
| taskId | INT → Tasks(id) |
| userId | INT → Users(id) |
| content | TEXT |
| createdAt | TIMESTAMP |

---

## 6. Проектирование API

### REST API Endpoints

**Аутентификация**

| Метод | Endpoint | Описание |
|-------|----------|----------|
| POST | /api/auth/login | Вход |
| POST | /api/auth/register | Регистрация |

**Проекты**

| Метод | Endpoint | Описание |
|-------|----------|----------|
| GET | /api/projects | Список моих проектов |
| POST | /api/projects | Создать проект |
| PUT | /api/projects/:id | Обновить |
| DELETE | /api/projects/:id | Удалить |
| POST | /api/projects/:id/members | Добавить участника |

**Задачи**

| Метод | Endpoint | Описание |
|-------|----------|----------|
| GET | /api/tasks?projectId=1 | Задачи проекта |
| POST | /api/tasks | Создать задачу |
| PATCH | /api/tasks/:id/status | Изменить статус |
| POST | /api/tasks/:id/comments | Добавить комментарий |

**Файлы**

| Метод | Endpoint | Описание |
|-------|----------|----------|
| POST | /api/projects/:id/files | Загрузить файл |
| GET | /api/files/:id | Скачать файл |
СЕРВЕРНАЯ ЧАСТЬ (BACKEND)
1. Краткое описание серверной части
Серверная часть реализована на Node.js с использованием фреймворка Express. Она обеспечивает:
	REST API для взаимодействия с клиентом (React)
	Аутентификацию и авторизацию (JWT + bcrypt)
	Работу с базой данных PostgreSQL через ORM Prisma
	Хранение файлов (отчёты, презентации)
	Разграничение ролей (student, teacher, admin)
Сервер принимает HTTP-запросы от фронтенда, обрабатывает их, взаимодействует с БД и возвращает JSON-ответы.
2. Описание основных функций сервера
2.1 Аутентификация (authController.js)
Функция	Метод	Эндпоинт	Параметры	Что делает	Возвращает
login	POST	/api/auth/login	{ email, password }	Проверяет email, сравнивает пароль (bcrypt), генерирует JWT	{ token, user: { id, email, fullName, role } }
register	POST	/api/auth/register	{ email, password, fullName, groupName }	Создаёт нового пользователя, хеширует пароль	{ token, user }
4.2 Проекты (projectController.js)
Функция	Метод	Эндпоинт	Параметры	Что делает	Возвращает
getMyProjects	GET	/api/projects	req.user.id (из токена)	Возвращает проекты, где пользователь — владелец или участник	Project[]
createProject	POST	/api/projects	{ title, description, deadline }	Создаёт новый проект, добавляет создателя как участника	Project
updateProject	PUT	/api/projects/:id	{ title, description, deadline, status }	Обновляет данные проекта (только владелец)	Project
deleteProject	DELETE	/api/projects/:id	-	Удаляет проект и все связанные задачи/файлы (только владелец)	{ success: true }
addMember	POST	/api/projects/:id/members	{ userId, roleInProject }	Добавляет участника в проект	{ success: true, member }
4.3 Задачи (taskController.js)
Функция	Метод	Эндпоинт	Параметры	Что делает	Возвращает
getTasksByProject	GET	/api/tasks?projectId=1	projectId (query)	Возвращает все задачи проекта	Task[]
createTask	POST	/api/tasks	{ projectId, title, assigneeId, deadline }	Создаёт новую задачу	Task
updateTaskStatus	PATCH	/api/tasks/:id/status	{ status }	Меняет статус задачи (todo/in_progress/done)	Task
deleteTask	DELETE	/api/tasks/:id	-	Удаляет задачу	{ success: true }
4.4 Оценки (gradeController.js) — для преподавателя
Функция	Метод	Эндпоинт	Параметры	Что делает	Возвращает
getSubmissions	GET	/api/grades/submissions	groupId (query)	Возвращает список сданных работ по группе	Submission[]
setGrade	POST	/api/grades/:taskId	{ grade, comment }	Выставляет оценку и комментарий к задаче	{ success: true, grade }

Схема взаимодействия клиента и сервера
 
3. Примеры запросов и ответов сервера
Пример 1: Регистрация нового пользователя
Запрос 
POST /api/auth/register
Content-Type: application/json

{
  "email": "ivanov@student.ru",
  "password": "securePass123",
  "fullName": "Иван Иванов",
  "groupName": "ИС-21"
}
Ответ
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 15,
    "email": "ivanov@student.ru",
    "fullName": "Иван Иванов",
    "role": "student",
    "groupName": "ИС-21"
  }
}
Пример 2: Создание нового проекта
Запрос
POST /api/projects
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json

{
  "title": "Разработка мобильного приложения",
  "description": "Создать приложение для учёта привычек",
  "deadline": "2026-06-15"
}
Ответ
{
  "id": 42,
  "ownerId": 15,
  "title": "Разработка мобильного приложения",
  "description": "Создать приложение для учёта привычек",
  "deadline": "2026-06-15",
  "status": "active",
  "createdAt": "2026-04-21T10:30:00.000Z",
  "members": [
    {
      "userId": 15,
      "roleInProject": "leader"
    }
  ]
}
Пример 3: Получение списка проектов (Дашборд)
Запрос
GET /api/projects
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Ответ
{
  "projects": [
    {
      "id": 42,
      "title": "Разработка мобильного приложения",
      "deadline": "2026-06-15",
      "status": "active",
      "progress": 30,
      "tasksCount": 8,
      "completedTasks": 2
    },
    {
      "id": 43,
      "title": "Курсовая работа по БД",
      "deadline": "2026-05-20",
      "status": "active",
      "progress": 75,
      "tasksCount": 4,
      "completedTasks": 3
    }
  ]
}
Пример 4: Создание задачи в проекте
Запрос
POST /api/tasks
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json

{
  "projectId": 42,
  "title": "Спроектировать базу данных",
  "assigneeId": 16,
  "deadline": "2026-05-01"
}
Ответ
{
  "id": 101,
  "projectId": 42,
  "title": "Спроектировать базу данных",
  "assigneeId": 16,
  "assigneeName": "Петр Петров",
  "deadline": "2026-05-01",
  "status": "todo",
  "grade": null,
  "createdAt": "2026-04-21T11:00:00.000Z"
}
Пример 5: Изменение статуса задачи (Kanban)
Запрос
PATCH /api/tasks/101/status
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json

{
  "status": "in_progress"
}
Ответ
{
  "success": true,
  "task": {
    "id": 101,
    "title": "Спроектировать базу данных",
    "status": "in_progress",
    "updatedAt": "2026-04-21T11:30:00.000Z"
  }
}
Пример 6: Выставление оценки (для преподавателя)
Запрос:
POST /api/grades/101
Authorization: Bearer eyJhbGciOiJIUzI1NiIs... (role: teacher)
Content-Type: application/json

{
  "grade": 85,
  "comment": "Хорошая работа, но добавьте нормализацию"
}
Ответ 
json
{
  "success": true,
  "grade": {
    "taskId": 101,
    "grade": 85,
    "comment": "Хорошая работа, но добавьте нормализацию",
    "gradedBy": "Преподаватель Смирнова А.И.",
    "gradedAt": "2026-04-21T14:20:00.000Z"
  }
}

КЛИЕНТСКАЯ ЧАСТЬ (FRONTEND)
1. Краткое описание клиентской части
Клиентская часть реализована на React с использованием Vite как сборщика. Для стилизации используется Material UI (MUI) — библиотека готовых компонентов, обеспечивающая современный и адаптивный дизайн.
Основные возможности клиентской части:
Функция	Описание
Адаптивный дизайн	Корректно работает на ПК, планшетах и смартфонах
Аутентификация	Вход и регистрация с сохранением JWT токена
Дашборд	Отображение всех проектов пользователя
Управление проектами	Создание, редактирование, просмотр проектов
Управление задачами	Создание, редактирование, изменение статуса (Kanban)
Загрузка файлов	Прикрепление отчётов и материалов к проектам
 Комментарии	Обсуждение задач внутри проекта
Журнал оценок	Просмотр оценок (студент) / выставление оценок (преподаватель)
 Структура
client/
├── public/
│   └── index.html
│
├── src/
│   ├── assets/               # Изображения, иконки
│   │   └── logo.png
│   │
│   ├── components/           # Переиспользуемые компоненты
│   │   ├── Layout/
│   │   │   ├── Header.jsx    # Верхняя панель (навигация)
│   │   │   ├── Sidebar.jsx   # Боковое меню
│   │   │   └── Footer.jsx
│   │   ├── ProjectCard.jsx   # Карточка проекта (на дашборде)
│   │   ├── TaskCard.jsx      # Карточка задачи (на Kanban)
│   │   ├── Comment.jsx       # Комментарий к задаче
│   │   └── FileUpload.jsx    # Компонент загрузки файлов
│   │
│   ├── pages/                # Страницы приложения
│   │   ├── LoginPage.jsx     # Страница входа
│   │   ├── RegisterPage.jsx  # Страница регистрации
│   │   ├── DashboardPage.jsx # Дашборд (список проектов)
│   │   ├── ProjectPage.jsx   # Страница конкретного проекта
│   │   ├── KanbanPage.jsx    # Доска задач (Kanban)
│   │   ├── CalendarPage.jsx  # Календарь дедлайнов
│   │   ├── GradesPage.jsx    # Журнал оценок
│   │   └── ProfilePage.jsx   # Профиль пользователя
│   │
│   ├── services/             # API-запросы к серверу
│   │   ├── api.js            # Базовый axios-клиент
│   │   ├── authService.js    # Логин, регистрация
│   │   ├── projectService.js # CRUD проектов
│   │   ├── taskService.js    # CRUD задач
│   │   ├── fileService.js    # Загрузка файлов
│   │   └── gradeService.js   # Работа с оценками
│   │
│   ├── store/                # Redux Toolkit (состояние)
│   │   ├── store.js
│   │   ├── authSlice.js      # Пользователь, токен
│   │   ├── projectSlice.js   # Список проектов
│   │   └── taskSlice.js      # Список задач
│   │
│   ├── hooks/                # Кастомные хуки
│   │   ├── useAuth.js
│   │   └── useProjects.js
│   │
│   ├── utils/                # Вспомогательные функции
│   │   ├── formatDate.js
│   │   └── validateForm.js
│   │
│   ├── App.jsx               # Главный компонент (роутинг)
│   ├── main.jsx              # Точка входа
│   └── index.css             # Глобальные стили
│
├── .env                      # Переменные окружения
├── .gitignore
├── index.html
├── package.json
└── vite.config.js
Схема навигации
 
Таблица маршрутов (React Router)
Путь	Компонент	Доступ	Описание
/login	LoginPage	Публичный	Страница входа
/register	RegisterPage	Публичный	Страница регистрации
/dashboard	DashboardPage	Приватный	Главная после входа
/projects/:id	ProjectPage	Приватный	Детали проекта
/projects/:id/kanban	KanbanPage	Приватный	Доска задач
/calendar	CalendarPage	Приватный	Календарь дедлайнов
/grades	GradesPage	Приватный	Журнал оценок
/profile	ProfilePage	Приватный	Настройки профиля
Основные экраны и блоки
Страница входа 
 
	Поле Email — ввод email
	Поле Пароль — ввод пароля (тип password)
	Кнопка ВОЙТИ — отправка запроса на сервер
	Ссылка Регистрация — переход на страницу регистрации
Дашборд
 
	[+ НОВЫЙ ПРОЕКТ] — открывает модальное окно создания проекта
	[Открыть] на карточке — переход на страницу проекта
	Виджет Дедлайны на неделю — список ближайших дедлайнов
	Аватар пользователя (вверху справа) — переход в профиль

Страница проекта
 
	ЗАДАЧИ — список задач (таблица)
	ФАЙЛЫ — загруженные файлы, кнопка загрузки
	УЧАСТНИКИ — список команды, добавление участников
	ОБСУЖДЕНИЕ — чат/комментарии по проекту

Доска Kanban
 
	Drag-and-Drop — перетаскивание карточек между колонками
	Каждая карточка содержит: название, ответственного, дедлайн
	Кнопка [+ ДОБАВИТЬ] — создание новой задачи

Создание проекта
// Компонент CreateProjectModal.jsx
const CreateProjectModal = ({ open, onClose }) => {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [deadline, setDeadline] = useState('');
  
  const handleSubmit = async () => {
    const response = await projectService.createProject({
      title, description, deadline
    });
    onClose();
    // Обновление списка проектов
  };
  
  return (
    <Modal open={open}>
      <input placeholder="Название проекта" onChange={e => setTitle(e.target.value)} />
      <textarea placeholder="Описание" onChange={e => setDescription(e.target.value)} />
      <input type="date" onChange={e => setDeadline(e.target.value)} />
      <button onClick={handleSubmit}>Создать</button>
    </Modal>
  );
};
Создание задач на Канбан
// Компонент CreateTaskForm.jsx
const CreateTaskForm = ({ projectId, onSuccess }) => {
  const [title, setTitle] = useState('');
  const [assigneeId, setAssigneeId] = useState('');
  const [deadline, setDeadline] = useState('');
  
  const handleSubmit = async () => {
    await taskService.createTask({
      projectId,
      title,
      assigneeId,
      deadline
    });
    onSuccess();
  };
  
  return ( /* форма */ );
};
Запрос клиента к серверу
// services/api.js
import axios from 'axios';

const API_URL = 'http://localhost:5000/api';

const api = axios.create({
  baseURL: API_URL,
  headers: { 'Content-Type': 'application/json' }
});

// Добавление токена в каждый запрос
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
сервис проектов
// services/projectService.js
import api from './api';

export const projectService = {
  // Получить все проекты пользователя
  getMyProjects: () => api.get('/projects'),
  
  // Создать проект
  createProject: (data) => api.post('/projects', data),
  
  // Обновить проект
  updateProject: (id, data) => api.put(`/projects/${id}`, data),
  
  // Удалить проект
  deleteProject: (id) => api.delete(`/projects/${id}`),
  
  // Добавить участника
  addMember: (projectId, userId, role) => 
    api.post(`/projects/${projectId}/members`, { userId, roleInProject: role })
};
Сервис задач
// services/taskService.js
import api from './api';

export const taskService = {
  // Получить задачи проекта
  getTasksByProject: (projectId) => api.get(`/tasks?projectId=${projectId}`),
  
  // Создать задачу
  createTask: (data) => api.post('/tasks', data),
  
  // Обновить статус задачи
  updateTaskStatus: (taskId, { status }) => 
    api.patch(`/tasks/${taskId}/status`, { status }),
  
  // Удалить задачу
  deleteTask: (taskId) => api.delete(`/tasks/${taskId}`)
};
Примеры запросов из клиента
Пример 1: Авторизация
// При клике на кнопку "Войти"
const handleLogin = async () => {
  const response = await authService.login({ email, password });
  localStorage.setItem('token', response.data.token);
  localStorage.setItem('user', JSON.stringify(response.data.user));
  navigate('/dashboard');
};
Пример 2. Создание проекта
// При отправке формы создания проекта
const handleCreateProject = async (formData) => {
  try {
    const response = await projectService.createProject(formData);
    dispatch(addProject(response.data)); // Redux store
    showSnackbar('Проект создан!', 'success');
    onClose();
  } catch (error) {
    showSnackbar('Ошибка: ' + error.message, 'error');
  }
};
