# SPEC.md — Декларация 910: Налоговое Приложение для ИП

## 1. Концепция и Видение

**"Мой Налоговый Консультант"** — премиальное мобильное приложение для казахстанских индивидуальных предпринимателей. Не пугающая бюрократия, а ясный пошаговый путь к заполнению декларации с полной прозрачностью расчётов и детализацией КБК.

Визуальный стиль: **Financial Architect** — глубокие синие тона, типографика Manrope/Inter, никаких линий — только тональные сдвиги. Приложение создаёт ощущение личного финансового советника в кармане.

---

## 2. Дизайн-система

### Цветовая палитра
| Роль | HEX | Использование |
|------|-----|--------------|
| Primary | `#000666` | Заголовки, акценты |
| Primary Container | `#1a237e` | Градиенты, карточки |
| Surface | `#f8f9fa` | Фон приложения |
| Surface Container Low | `#f3f4f5` | Поля ввода |
| Surface Container Lowest | `#ffffff` | Карточки |
| Tertiary (Success) | `#005312` | Статус "Оплачено" |
| Tertiary Fixed | `#a3f69c` | Прогресс-бары |
| Error | `#ba1a1a` | Ошибки валидации |
| On Surface | `#191c1d` | Основной текст |
| On Surface Variant | `#454652` | Вторичный текст |

### Типографика
- **Manrope** (800, 700) — заголовки, суммы, дисплеи
- **Inter** (500, 600) — тело текста, метки, кнопки
- Шкала: `display-lg` (48px) → `headline-sm` (24px) → `body-sm` (14px) → `label-xs` (11px)

### Пространство
- Отступы страниц: 24px
- Внутренние отступы карточек: 20px
- Межсекционные отступы: 32px
- Радиусы: `xl` (12px), `lg` (8px), `md` (6px)

---

## 3. Экраны и Навигация

### 3.1 Выбор языка (Splash/Start)
- Две карточки: 🇷🇺 Русский / 🇰🇿 Қазақша
- При нажатии — анимация scale + переход к авторизации
- Сохранение языка в localStorage

### 3.2 Авторизация
**SMS-авторизация:**
- Ввод номера телефона (формат +7XXXXXXXXXX)
- Получение кода, ввод 6-значного кода
- Таймер повторной отправки (60 сек)

**ЭЦП-авторизация:**
- Кнопка "Войти с ЭЦП"
- Выбор файла ключа (.p12, .key, storage.p12)
- Ввод пароля от ключа
- В демо-режиме — любой файл принимается

**Подготовка к NCALayer (future):**
```
// Архитектура для интеграции с NCALayer
interface ECPSigner {
  selectKey(): Promise<KeyInfo>
  sign(data: ArrayBuffer): Promise<Signature>
  getCertificate(): Promise<X509Cert>
}

// Реализация через WebExtension API
class NCALayerSigner implements ECPSigner {
  private ncalayer: NCALayerWebSocket
  
  async sign(data: ArrayBuffer): Promise<Signature> {
    const response = await this.ncalayer.send({
      type: 'cms-sign',
      data: base64Encode(data),
      plugin: 'kz.gov.pki.cms'
    })
    return base64Decode(response.signature)
  }
}
```

### 3.3 Главный экран (Dashboard)
- Приветственная карточка: ИП "Бизнес Ко", БИН 940312300456
- Кнопка "Новая декларация" (основная CTA)
- Кнопка "Мои декларации" (вторичная)
- Статистика: Всего подано, Следующий срок

### 3.4 Мои декларации
- Список карточек деклараций
- Каждая карточка: Период, Дата, Статус, Сумма
- Статусы: "Подана" (зелёный), "Черновик" (серый), "Ошибка" (красный)
- Фильтр по статусу и периоду

### 3.5 Создание декларации — Выбор режима
- Карточки: "Упрощённая декларация (910)" / "Общеустановленный режим"
- Краткое описание каждого режима
- Переход к форме

### 3.6 Форма 910 (Упрощённая декларация)

**Входные данные:**
| Поле | Тип | Описание |
|------|-----|----------|
| Период | dropdown | 1 п/г 2025, 2 п/г 2025, 1 п/г 2026... |
| Доход (безналичный) | number | Доходы от безналичных расчётов |
| Доход (наличный) | number | Доходы от наличных расчётов |
| Количество работников | number | 0-30 |
| Зарплата (если есть) | number | Среднемесячная на 1 работника |

**Автоматический расчёт:**
```
Итого доход = Безналичный + Наличный
ИПН = Итого × 1.5%
Социальный налог = Итого × 1.5% - Соц. отчисления
Итого налог = ИПН + Соц. налог
```

### 3.7 Форма ОУР (Общеустановленный режим)

**Входные данные для ИП:**
| Поле | Тип | Описание |
|------|-----|----------|
| Год | dropdown | 2024, 2025, 2026 |
| Период | dropdown | 1 квартал - 4 квартал |
| Общий доход за период | number | Сумма дохода |
| Расходы (если есть) | number | Документально подтверждённые |
| Количество работников | number | Без ограничений |
| Средняя з/п работника | number | Для расчёта СО, ОПВ |

**Автоматический расчёт (ОУР):**
```
// ИПН за себя
Доход для ИПН = Общий доход - Расходы
Вычет МЗП = 85,000 × 12 = 1,020,000
База ИПН = Доход для ИПН - Вычет МЗП - ОПВ за себя - СО за себя
ИПН за себя = База ИПН × 10%

// Социальный налог за себя
СН за ИП = 2 × МРП × кол-во кварталов = 2 × 3,932 × 1 = 7,864

// ОПВ за себя (ежемесячно, за квартал × 3)
ОПВ за себя = max(85,000, доход) × 10% × 3

// СО за себя (ежемесячно, min 4,250, max 29,750)
СО за себя = max(4,250, доход × 5%) × 3

// ОПВ за работников (10%, min 1 МЗП, max 50 МЗП)
ОПВ за раб. = min(max(зарплата, 85,000) × 10%, 425,000)

// СО за работников (5%, min 4,250, max вычеты)
СО за раб. = min(max(зарплата × 5%, 4,250), 29,750)

// ОПВР за работников (2.5%)
ОПВР = зарплата × 2.5%

// ОСМС отчисления за работников (3%)
ОСМС = зарплата × 3%

// ВОСМС за работников (2%, или фикс. 5,950 за ИП)
ВОСМС = зарплата × 2%
```

### 3.8 Валидация декларации

**Уровни проверки:**
```
interface ValidationResult {
  level: 'error' | 'warning' | 'info'
  code: string
  message: { ru: string; kk: string }
  field?: string
  suggestion?: string
}

// Демо-валидатор (реализует логику КГД)
class DeclarationValidator {
  
  validateBasic(formData: FormData): ValidationResult[] {
    const errors: ValidationResult[] = []
    
    // Проверка обязательных полей
    if (!formData.period) {
      errors.push({
        level: 'error',
        code: 'REQ_001',
        message: {
          ru: 'Не указан налоговый период',
          kk: 'Салық кезеңі көрсетілмеген'
        },
        field: 'period'
      })
    }
    
    // Проверка лимита дохода для Упрощёнки
    if (formData.regime === 'simplified') {
      const maxIncome = 24038 * 3932 // МРП 2025
      if (formData.income > maxIncome) {
        errors.push({
          level: 'error',
          code: 'LIM_001',
          message: {
            ru: `Доход превышает лимит ${this.formatMoney(maxIncome)}`,
            kk: `Табыс шегі ${this.formatMoney(maxIncome)} асып түсті`
          },
          field: 'income'
        })
      }
    }
    
    // Проверка количества работников
    if (formData.employees > 30 && formData.regime === 'simplified') {
      errors.push({
        level: 'error',
        code: 'EMP_001',
        message: {
          ru: 'Превышен лимит работников (максимум 30 для упрощёнки)',
          kk: 'Жұмысшылар лимиті асып түсті (жеңілдетілген үшін максимум 30)'
        },
        field: 'employees'
      })
    }
    
    // Проверка МЗП для ОПВ
    if (formData.regime === 'our') {
      const minWage = 85000
      if (formData.salary < minWage) {
        errors.push({
          level: 'warning',
          code: 'WRN_001',
          message: {
            ru: `Зарплата ниже МЗП (${this.formatMoney(minWage)})`,
            kk: `Жалақы МАО-дан төмен (${this.formatMoney(minWage)})`
          },
          field: 'salary'
        })
      }
    }
    
    return errors
  }
  
  validateCalculations(formData: FormData, expected: TaxCalculation): ValidationResult[] {
    const errors: ValidationResult[] = []
    
    // Проверка сумм
    if (expected.totalTax < 0) {
      errors.push({
        level: 'error',
        code: 'CAL_001',
        message: {
          ru: 'Итоговая сумма налога не может быть отрицательной',
          kk: 'Салықтың жалпы сомасы теріс болмайды'
        }
      })
    }
    
    return errors
  }
}
```

### 3.9 Экран КБК для оплаты

**Структура:**
```
┌─────────────────────────────────────┐
│  СПИСОК ПЛАТЕЖЕЙ                   │
├─────────────────────────────────────┤
│  ┌─────────────────────────────┐   │
│  │ ИПН (КБК 101202)            │   │
│  │ 225,000 ₸                    │   │
│  │ ████████████░░░░░░░  50%    │   │
│  └─────────────────────────────┘   │
│  ┌─────────────────────────────┐   │
│  │ Социальный налог (КБК 103101)│   │
│  │ 227,840 ₸                    │   │
│  │ ████████████████░░░  50%     │   │
│  └─────────────────────────────┘   │
│                                     │
│  ИТОГО: 452,840 ₸                  │
│                                     │
│  [💳 Оплатить все]                  │
│  [📤 Экспорт] [📤 Поделиться]      │
└─────────────────────────────────────┘
```

**КБК для Упрощёнки (910):**
| КБК | Наименование | Ставка |
|-----|--------------|--------|
| 101202 | ИПН с доходов ИП | 1.5% |
| 103101 | Социальный налог | 1.5% - СО |

**КБК для ОУР:**
| КБК | Наименование | Примечание |
|-----|--------------|------------|
| 101201 | ОПВ (пенсионные взносы) | 10% от дохода |
| 101202 | ИПН (доход ИП) | 10% от базы |
| 102102 | Социальные отчисления | 5% от дохода |
| 103101 | Социальный налог | 2 МРП (ИП) + 1 МРП (раб.) |
| 103102 | Соц. налог с работников | 1 МРП × кол-во |
| 901101 | ОПВР (военнослужащие) | 2.5% |
| 921101 | Отчисления ОСМС | 3% от з/п |
| 921102 | Взносы ВОСМС | 2% от з/п |

### 3.10 Экран подтверждения
- Чекмарк-анимация
- ID декларации
- QR-код для проверки
- Кнопки: "Скачать PDF", "Просмотр декларации"

### 3.11 Загрузка и парсинг банковской выписки

**Поддерживаемые форматы:**
| Банк | Форматы | Примечание |
|------|---------|-----------|
| Kaspi Pay | CSV, Excel, PDF, 1C | Экспорт через мобильное приложение |
| Halyk Bank | CSV, Excel, PDF | Onlinebank |
| Jusan Bank | CSV, Excel, PDF | Мобильное приложение |
| Все банки | CSV | Универсальный формат |

**Экран предпросмотра выписки:**
```
┌─────────────────────────────────────┐
│  ПРЕДПРОСМОТР ВЫПИСКИ            │
├─────────────────────────────────────┤
│  Банк: Kaspi Pay                   │
│  Период: 01.01.2025 - 30.06.2025 │
│  ─────────────────────────────────  │
│                                     │
│  📊 СВОДКА                         │
│  ─────────────────────────────────  │
│  Всего поступлений:    15,000,000 ₸│
│  Безналичные:         10,500,000 ₸│
│  Наличные:             4,500,000 ₸ │
│  ─────────────────────────────────  │
│                                     │
│  📅 ПО МЕСЯЦАМ                     │
│  Январь    ████████████  2,500,000 │
│  Февраль   ██████████░░  2,000,000  │
│  ...                             │
│                                     │
│  [✓] Подтвердить]  [✎ Редактировать]│
└─────────────────────────────────────┘
```

**Парсинг PDF (Kaspi Pay):**
```javascript
// Использование pdf.js для извлечения текста из PDF
async function parseKaspiPdf(file: File): Promise<Transaction[]> {
  const pdfData = await loadPdf(file)
  const text = await extractText(pdfData)
  
  // Регулярные выражения для извлечения данных
  const patterns = {
    date: /(\d{2}\.\d{2}\.\d{4})/g,
    amount: /([\d\s]+)\s*₸/g,
    type: /(поступление|списание|перевод)/gi
  }
  
  return extractTransactions(text, patterns)
}
```

### 3.12 Управление данными организации

**Автозаполнение из eGov.kz:**
```javascript
// API для получения данных ИП по БИН
interface EGovAPI {
  // Получить данные юр. лица / ИП
  getEntityByBIN(bin: string): Promise<EntityData>
  
  // Проверить статус контрагента
  checkCounterparty(bin: string): Promise<CounterpartyStatus>
}

interface EntityData {
  bin: string
  nameRu: string
  nameKz: string
  address: string
  oked: string
  registrationDate: string
  status: 'active' | 'liquidated' | 'suspended'
  headName?: string
  taxAuthority?: string
}

// Пример использования
async function fillOrganizationData(bin: string) {
  const response = await fetch(
    `https://data.egov.kz/api/v4/ds/binary?bin=${bin}`
  )
  const data = await response.json()
  
  // Автозаполнение формы
  document.getElementById('orgName').value = data.nameRu
  document.getElementById('orgAddress').value = data.address
  document.getElementById('orgOKED').value = data.oked
}
```

**Ручное редактирование:**
- Возможность корректировки любого поля
- Валидация изменённых данных
- Сохранение пользовательских корректировок

### 3.13 Управление ЭЦП и история подписаний

**Функции:**
```
┌─────────────────────────────────────┐
│  🔐 УПРАВЛЕНИЕ ЭЦП                │
├─────────────────────────────────────┤
│                                     │
│  📜 МОИ КЛЮЧИ                      │
│  ┌─────────────────────────────┐   │
│  │ 🔒 ЭЦП Физического лица    │   │
│  │    Действителен до: 01.2027 │   │
│  │    [Просмотр] [Удалить]     │   │
│  └─────────────────────────────┘   │
│  ┌─────────────────────────────┐   │
│  │ 🏢 ЭЦП Юридического лица   │   │
│  │    Действителен до: 06.2025 │   │
│  │    ⚠️ Срок истекает!       │   │
│  │    [Продлить] [Просмотр]    │   │
│  └─────────────────────────────┘   │
│                                     │
│  📋 ИСТОРИЯ ПОДПИСАНИЙ             │
│  ─────────────────────────────────  │
│  15.02.2026  ФНО 910.00    ✓     │
│  14.01.2026  ФНО 200.00    ✓     │
│  10.12.2025  Договор       ✓     │
│                                     │
│  [Загрузить новый ключ]            │
└─────────────────────────────────────┘
```

**Интерфейс:**
```javascript
interface ECPManager {
  // Список загруженных ключей
  getStoredKeys(): Promise<ECPKey[]>
  
  // Добавить ключ
  addKey(file: File, password: string): Promise<ECPKey>
  
  // Проверить срок действия
  checkExpiry(key: ECPKey): ExpiryStatus
  
  // История подписаний
  getSigningHistory(): Promise<SigningRecord[]>
}

interface SigningRecord {
  id: string
  documentType: string
  documentId: string
  signedAt: Date
  status: 'signed' | 'verified' | 'expired'
  signatureHash: string
}
```

### 3.14 Валидация и подтверждение изменений

**Типы проверок:**
```javascript
interface ValidationCheck {
  type: 'error' | 'warning' | 'info'
  code: string
  field?: string
  message: LocalizedString
  suggestion?: string
  autoFixable: boolean
}

// Система проверки
class DeclarationValidator {
  
  // Базовые проверки
  validateBasic(formData: FormData): ValidationCheck[]
  
  // Проверка сумм
  validateCalculations(formData: FormData): ValidationCheck[]
  
  // Проверка лимитов
  validateLimits(formData: FormData): ValidationCheck[]
  
  // Проверка обязательных приложений
  validateRequiredAttachments(formData: FormData): ValidationCheck[]
  
  // Полный цикл валидации
  validateAll(formData: FormData): ValidationResult
}

interface ValidationResult {
  isValid: boolean
  errors: ValidationCheck[]
  warnings: ValidationCheck[]
  info: ValidationCheck[]
  canSubmit: boolean
  estimatedProcessingTime?: string
}
```

**Экран подтверждения:**
```
┌─────────────────────────────────────┐
│  ✓ ПРОВЕРКА ЗАВЕРШЕНА             │
├─────────────────────────────────────┤
│                                     │
│  ┌─────────────────────────────┐   │
│  │ ✓ Обязательные поля         │   │
│  │ ✓ Расчёты корректны         │   │
│  │ ⚠️ Превышен лимит дохода   │   │
│  │   (можно игнорировать)      │   │
│  └─────────────────────────────┘   │
│                                     │
│  Декларация готова к отправке      │
│                                     │
│  [Подписать и отправить]            │
│  [Сохранить как черновик]           │
│  [Исправить ошибки]                │
└─────────────────────────────────────┘
```

### 3.15 История поданных деклараций (eGov)

**Интеграция с порталом:**
```javascript
interface DeclarationHistoryAPI {
  // Получить список всех деклараций
  getAllDeclarations(params: HistoryQuery): Promise<PaginatedResult<Declaration>>
  
  // Получить декларацию по ID
  getDeclaration(id: string): Promise<Declaration>
  
  // Статус обработки
  getProcessingStatus(submissionId: string): Promise<ProcessingStatus>
  
  // Скачать PDF
  downloadDeclarationPdf(id: string): Promise<Blob>
}

interface HistoryQuery {
  formType?: string       // '910.00', '200.00', etc.
  year?: number
  period?: 'first_half' | 'second_half'
  status?: 'accepted' | 'pending' | 'rejected'
  page?: number
  pageSize?: number
}
```

**Экран истории:**
```
┌─────────────────────────────────────┐
│  📋 ИСТОРИЯ ДЕКЛАРАЦИЙ            │
├─────────────────────────────────────┤
│  [Фильтры ▼]  [Поиск...]          │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ ФНО 910.00 - 2 п/г 2025   │   │
│  │ 15.02.2026 • 452,840 ₸    │   │
│  │ ✓ Принята КГД              │   │
│  │ [Скачать] [Детали]         │   │
│  └─────────────────────────────┘   │
│  ┌─────────────────────────────┐   │
│  │ ФНО 910.00 - 1 п/г 2025   │   │
│  │ 14.08.2025 • 380,000 ₸    │   │
│  │ ✓ Принята КГД              │   │
│  └─────────────────────────────┘   │
│  ┌─────────────────────────────┐   │
│  │ ФНО 910.00 - 2 п/г 2024   │   │
│  │ 14.02.2025 • 290,000 ₸    │   │
│  │ ✓ Принята КГД              │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

---

## 4. Техническая архитектура

**Парсинг выписок:**
```javascript
// Универсальный интерфейс парсера
interface BankStatementParser {
  detectBankFormat(content: string): BankType
  parseTransactions(content: string): Transaction[]
  categorizeIncome(transactions: Transaction[]): IncomeSummary
}

// Категоризация транзакций
interface Transaction {
  date: Date
  amount: number
  description: string
  type: 'credit' | 'debit'
  counterparty?: string
}

interface IncomeSummary {
  totalIncome: number        // Общий доход
  cashIncome: number         // Наличные поступления
  nonCashIncome: number      // Безналичные поступления
  byMonth: MonthlyBreakdown[] // По месяцам
  transactions: Transaction[]  // Детализация
}

// Поддерживаемые банки
enum BankType {
  KASPI = 'kaspi',
  HALYK = 'halyk',
  JUSAN = 'jusan',
  GENERIC_CSV = 'generic_csv',
  UNKNOWN = 'unknown'
}
```

**Логика определения дохода:**
```
Поступления (credit) - это доход ИП
Списания (debit) - это расходы (не учитываются в декларации 910)
Возвраты - исключаются из дохода
Переводы между своими счетами - исключаются
```

### 3.12 Экран выбора файла выписки

```
┌─────────────────────────────────────┐
│  ЗАГРУЗКА ВЫПИСКИ                 │
├─────────────────────────────────────┤
│                                     │
│  ┌─────────────────────────────┐   │
│  │  📄 Перетащите файл сюда   │   │
│  │       или нажмите           │   │
│  └─────────────────────────────┘   │
│                                     │
│  Поддерживаемые форматы:            │
│  • CSV (Kaspi, Halyk, Jusan)       │
│  • Excel (.xlsx, .xls)             │
│  • PDF (Kaspi Pay)                  │
│                                     │
│  ─────────── или ───────────       │
│                                     │
│  [📱 Загрузить из Kaspi]           │
│  [🏦 Загрузить из Halyk]           │
│  [💳 Загрузить из Jusan]           │
│                                     │
│  ─────────────────────────────      │
│  💡 Выписка должна содержать       │
│     операции за полугодие           │
└─────────────────────────────────────┘
```

---

## 4. Техническая архитектура

### 4.1 Стек
- **Frontend**: Vanilla JS + HTML + CSS (single file)
- **Стилизация**: Tailwind CSS (CDN) + custom CSS variables
- **Шрифты**: Google Fonts (Manrope, Inter)
- **Иконки**: Material Symbols Outlined
- **Хранение**: localStorage (IndexedDB для больших объёмов)
- **PDF парсинг**: pdf.js (для выписок Kaspi PDF)

### 4.2 Структура данных

```javascript
// Модель декларации
interface Declaration {
  id: string
  createdAt: string
  updatedAt: string
  status: 'draft' | 'submitted' | 'error'
  regime: 'simplified' | 'our'
  period: {
    year: number
    half?: '1' | '2'
    quarter?: '1' | '2' | '3' | '4'
  }
  formData: {
    income?: number
    cashIncome?: number
    nonCashIncome?: number
    employees?: number
    salary?: number
    expenses?: number
  }
  calculations: TaxCalculation
  validationErrors: ValidationResult[]
  submittedAt?: string
}

// Модель пользователя
interface User {
  id: string
  name: string
  bin: string
  authMethod: 'sms' | 'ecp'
  phone?: string
}
```

### 4.3 Архитектура для NCALayer (Future)

```
┌─────────────────────────────────────────────────────────────┐
│                      WEB APPLICATION                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │   UI/UX     │  │  Business    │  │  Data Layer      │  │
│  │  Components │  │  Logic       │  │  (localStorage)  │  │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘  │
│         └─────────────────┼─────────────────┘            │
│                           │                                │
│                    ┌──────▼──────┐                        │
│                    │ Signer API  │  (Abstraction Layer)    │
│                    └──────┬──────┘                        │
└───────────────────────────┼─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐  ┌───────────────┐  ┌─────────────────┐
│ DemoSigner    │  │ NCALayerProxy │  │ eGovMobile SDK  │
│ (placeholder) │  │ (WebSocket)   │  │ (Native Bridge) │
└───────────────┘  └───────────────┘  └─────────────────┘
```

### 4.4 API-заглушки (для демо)

```javascript
// Симуляция отправки в КГД
async function submitToKGD(declaration: Declaration): Promise<SubmitResult> {
  await simulateNetworkDelay(1500)
  return {
    success: true,
    declarationId: `910-${Date.now()}`,
    status: 'accepted',
    timestamp: new Date().toISOString()
  }
}

// Симуляция проверки через КГД
async function verifyDeclaration(id: string): Promise<VerificationResult> {
  await simulateNetworkDelay(1000)
  return {
    status: 'verified',
    checkedAt: new Date().toISOString(),
    errors: []
  }
}
```

### 4.5 Интеграции с внешними API

```javascript
// ===== eGov.kz Open Data API =====
const EGOV_API = {
  baseUrl: 'https://data.egov.kz/api/v4',
  
  // Поиск организации по БИН
  async getEntityByBIN(bin: string): Promise<EntityData> {
    const response = await fetch(
      `${this.baseUrl}/ds/binary?bin=${bin}`
    )
    return response.json()
  },
  
  // Поиск по открытым данным
  async searchDataset(query: object): Promise<any[]> {
    const response = await fetch(
      `${this.baseUrl}/dataset?source=${JSON.stringify(query)}`
    )
    return response.json()
  }
}

// ===== API для проверки контрагента =====
const COUNTERPARTY_API = {
  // Бесплатный поиск по БИН/ИИН
  async search(query: string): Promise<SearchResult[]> {
    // Используем открытые источники
    const response = await fetch(
      `https://stat.gov.kz/api/ds?query=${query}`
    )
    return response.json()
  }
}

// ===== Истоия деклараций из eGov =====
const DECLARATION_HISTORY_API = {
  // Заглушка для демо - в реальности требует авторизации
  async getHistory(params: HistoryQuery): Promise<PaginatedResult> {
    // Симуляция
    return simulateHistoryResponse(params)
  }
}
```

---

## 5. Дополнительные API и интеграции (для будущей версии)

### 5.1 Рекомендуемые интеграции

| Сервис | Назначение | Приоритет | Статус |
|--------|-----------|-----------|--------|
| **eGov.kz Open Data** | Данные организации по БИН | Высокий | Готово |
| **Kaspi Pay API** | Выписки автоматически | Средний | Требует партнёрства |
| **Smart Bridge** | Отправка в КГД | Высокий | Требует доступа |
| **SIGEX API** | Упрощённое подписание | Средний | Требует подписки |
| **Электронный документооборот** | Интеграция с бухгалтерией | Низкий | По запросу |

### 5.2 Потенциальные улучшения

```javascript
// ===== Smart Bridge API (отправка в КГД) =====
const SMART_BRIDGE_API = {
  baseUrl: 'https://sb.egov.kz',
  
  // Авторизация
  async authenticate(ecpKey: ECPKey): Promise<AuthToken> {
    const signed = await signData(ecpKey, { service: 'KGD-FNO' })
    return fetch(`${this.baseUrl}/auth/token`, {
      method: 'POST',
      body: JSON.stringify(signed)
    }).then(r => r.json())
  },
  
  // Отправка формы налоговой отчётности
  async submitFNO(token: string, declaration: Declaration): Promise<SubmitResult> {
    return fetch(`${this.baseUrl}/api/fno/submit`, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${token}` },
      body: JSON.stringify(declaration)
    }).then(r => r.json())
  },
  
  // Проверка статуса
  async getStatus(submissionId: string): Promise<ProcessingStatus> {
    return fetch(`${this.baseUrl}/api/fno/status/${submissionId}`)
      .then(r => r.json())
  }
}

// ===== Kaspi API (автоимпорт выписок) =====
const KASPI_API = {
  // OAuth авторизация
  async authorize(): Promise<OAuthToken> {
    // Редирект на Kaspi OAuth
    window.location.href = 'https://kaspi.kz/oauth/authorize?...'
    return handleCallback()
  },
  
  // Получение транзакций
  async getTransactions(token: string, params: TransactionQuery): Promise<Transaction[]> {
    return fetch('https://api.kaspi.kz/v1/transactions', {
      headers: { 'Authorization': `Bearer ${token}` }
    }).then(r => r.json())
  },
  
  // Категоризация для налоговой
  categorizeForTax(transactions: Transaction[]): IncomeSummary {
    return {
      totalIncome: transactions
        .filter(t => t.type === 'credit' && !t.isRefund)
        .reduce((sum, t) => sum + t.amount, 0),
      byMonth: groupByMonth(transactions)
    }
  }
}

// ===== Календарь сроков =====
const DEADLINE_CALENDAR = {
  // Сроки подачи декларации 910
  deadlines: [
    { period: '1-2025', submitBy: '2025-08-15', payBy: '2025-08-25' },
    { period: '2-2025', submitBy: '2026-02-16', payBy: '2026-02-25' },
    // ...
  ],
  
  // Push-уведомления (через Service Worker)
  async scheduleReminder(deadline: Date, message: string) {
    if ('Notification' in window) {
      const permission = await Notification.requestPermission()
      if (permission === 'granted') {
        new Notification('Напоминание о сроке', { body: message })
      }
    }
  },
  
  // Локальные напоминания
  checkUpcomingDeadlines() {
    const today = new Date()
    const upcoming = this.deadlines.filter(d => {
      const daysUntil = (new Date(d.submitBy) - today) / (1000 * 60 * 60 * 24)
      return daysUntil <= 7 && daysUntil > 0
    })
    return upcoming
  }
}

// ===== Biometric (Digital ID) авторизация =====
const BIOMETRIC_AUTH = {
  // Проверка через eGov Mobile
  async signWithDigitalID(documentHash: string): Promise<Signature> {
    // Использование QR-кода или диплинка
    const qrData = generateQRForSigning({
      action: 'sign',
      documentHash,
      callbackUrl: window.location.origin + '/auth/callback'
    })
    
    // Показываем QR пользователю
    showQRModal(qrData)
    
    // Ждём подтверждения от eGov Mobile
    return waitForSignature(qrData.sessionId)
  },
  
  // Проверка статуса подписания
  async checkSignatureStatus(sessionId: string): Promise<SignatureStatus> {
    return fetch(`${API_BASE}/signature/status/${sessionId}`)
      .then(r => r.json())
  }
}
```

---

## 6. Функциональность

### 6.1 Основные функции
- [x] Выбор языка (RU/KZ)
- [x] Авторизация (SMS / ЭЦП)
- [x] Переключение между режимами (ОУР / Упрощёнка)
- [x] Ввод данных декларации
- [x] Автоматический расчёт налогов
- [x] Детальный список КБК
- [x] Валидация формы
- [x] Сохранение черновиков
- [x] История деклараций
- [x] Экспорт в PDF (через браузер)
- [x] Подготовка к NCALayer
- [x] Загрузка банковской выписки (CSV, Excel, PDF)
- [x] Парсинг и категоризация транзакций
- [x] Предпросмотр и редактирование данных выписки
- [x] Автозаполнение из eGov.kz (по БИН)
- [x] Управление ключами ЭЦП
- [x] История подписаний документов
- [x] Проверка ошибок и валидация
- [x] Подтверждение перед отправкой
- [x] История поданных деклараций

### 6.2 Дополнительные функции (для будущей версии)
- [ ] Интеграция с NCALayer (WebExtension)
- [ ] Отправка через eGov Mobile
- [ ] Push-уведомления о сроках (Календарь)
- [ ] Интеграция с Kaspi API (автоимпорт)
- [ ] Smart Bridge API (отправка в КГД)
- [ ] Biometric Digital ID авторизация
- [ ] Генерация QR для оплаты
- [ ] Синхронизация с кабинетом КГД

---

## 6. Ключевые КБК и расчёты

### 6.1 Упрощённая декларация (910)

**Для ИП без работников:**
```
Доход = Безналичный + Наличный
ИПН = Доход × 1.5%
Соц. налог = Доход × 1.5% - 20,825 (мин. СО за себя)

Пример: Доход = 10,000,000
  ИПН = 150,000
  СО за себя = 20,825
  Соц. налог = 150,000 - 20,825 = 129,175
  ИТОГО = 279,175
```

**Для ИП с работниками:**
```
ИПН за ИП = Доход × 1.5%
ИПН за раб. = (ФОТ - ОПВ - СО - ВОСМС) × 10% - вычеты

Снижение на 1.5% если з/п ≥ 23 МРП (90,436)
```

### 6.2 Общеустановленный режим

**ИП за себя:**
```
ОПВ = min(max(доход × 10%, 85,000), 425,000)
СО = min(max(доход × 5%, 4,250), 29,750)
ВОСМС = 5,950 (фикс. в месяц)
ИПН = max(0, (доход - 1,020,000 - ОПВ - СО) × 10%)
СН = 7,864 (2 МРП за квартал)
```

**За работников (на 1 человека):**
```
ОПВ = min(max(зарплата × 10%, 85,000), 425,000)
ОПВР = зарплата × 2.5%
СО = min(max(зарплата × 5%, 4,250), 29,750)
ОСМС = зарплата × 3%
ВОСМС = зарплата × 2%
ИПН = max(0, (зарплата - ОПВ - СО - ВОСМС - вычеты) × 10%)
СН = 3,932 (1 МРП)
```

---

## 7. Ресурсы и МРП на 2025-2026

| Показатель | 2025 | 2026 |
|------------|-------|------|
| МРП | 3,932 ₸ | 4,325 ₸ |
| МЗП | 85,000 ₸ | 85,000 ₸ |
| 1 МЗП для ОПВ | 85,000 ₸ | 85,000 ₸ |
| 50 МЗП (max ОПВ) | 4,250,000 ₸ | 4,250,000 ₸ |
| Min СО (в мес) | 4,250 ₸ | 4,250 ₸ |
| Max СО (в мес) | 29,750 ₸ | 29,750 ₸ |
| Min ВОСМС (ИП) | 5,950 ₸ | 5,950 ₸ |

---

## 8. Сроки сдачи

| Период | Срок подачи | Срок оплаты |
|--------|-------------|-------------|
| 1 п/г 2025 | 15.08.2025 | 25.08.2025 |
| 2 п/г 2025 | 16.02.2026 | 25.02.2026 |
| 1 п/г 2026 | 17.08.2026 | 25.08.2026 |
| 2 п/г 2026 | 15.02.2027 | 25.02.2027 |
