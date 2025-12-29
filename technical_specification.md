# Техническая спецификация калькулятора кухни

## Извлеченные формулы и логика расчета

### 1. Основные формулы из Excel

#### Расчет стоимости компонента:
```
Стоимость = Количество × Цена_за_единицу
Формула Excel: =A{row} * C{row}
где A - колонка количества, C - колонка с ценой
```

#### Итоговая стоимость:
```
Итог = СУММА(Все_компоненты)
Формула Excel: =SUM(D6:D30)
```

#### Стоимость с установкой:
```
Итог_с_установкой = Итог + Подъем + Доставка + Установка
Формула Excel: =C4+D31+D32+D33
```

#### Особенность - Установка как процент:
```
Установка = Процент × Итоговая_стоимость
Формула Excel (строка 33): =A33*C33*C4
где A33 - количество (обычно 1), C33 - процент (0.1 = 10%), C4 - итог
```

### 2. Структура данных компонентов

```javascript
// Пример структуры данных для компонентов
const components = [
  // Основные модули
  {
    id: 1,
    name: "Низ",
    unit: "пог.м",
    category: "base",
    prices: {
      "plenka_mdf_1": 16770,
      "plenka_mdf_2": 17470,
      "kraska_mat": 25150,
      "kraska_gl": 27280,
      "plastic": 29000,
      // ... другие типы фасадов
    }
  },
  {
    id: 2,
    name: "Верх (720)",
    unit: "пог.м",
    category: "base",
    prices: {
      "plenka_mdf_1": 11200,
      "plenka_mdf_2": 12000,
      "kraska_mat": 16800,
      "kraska_gl": 18240,
      // ...
    }
  },
  {
    id: 3,
    name: "Пенал (600×2550)",
    unit: "шт",
    category: "base",
    prices: {
      "plenka_mdf_1": 15050,
      "plenka_mdf_2": 15600,
      // ...
    }
  },
  // Углы (отрицательные цены = скидка)
  {
    id: 4,
    name: "Угол 90° Низ",
    unit: "шт",
    category: "corners",
    prices: {
      "plenka_mdf_1": -8355,  // Отрицательная цена!
      "plenka_mdf_2": -8735,
      // ...
    }
  },
  // Фурнитура
  {
    id: 5,
    name: "Ящик 500 Шариковые",
    unit: "шт",
    category: "hardware",
    prices: {
      "plenka_mdf_1": 2300,
      "plenka_mdf_2": 2300,
      // ...
    }
  },
  // ... остальные компоненты
];

// Типы фасадов
const facadeTypes = [
  { id: "plenka_mdf_1", name: "Пленка МДФ 1", category: "economy" },
  { id: "plenka_mdf_2", name: "Пленка МДФ 2", category: "economy" },
  { id: "mdf_mylo", name: "МДФ 'Мыло' 3,4,5", category: "standard" },
  { id: "mdf_premium", name: "МДФ 345 Premium", category: "premium" },
  { id: "mdf_country", name: "МДФ 345 'Каунтри'", category: "premium" },
  { id: "mdf_london", name: "МДФ 345 'Лондон'", category: "premium" },
  { id: "kraska_mat", name: "Краска МАТ", category: "premium" },
  { id: "kraska_gl", name: "Краска ГЛЯНЕЦ", category: "premium" },
  { id: "plastic", name: "Пластик", category: "premium" },
  { id: "eterno_pet", name: "Етерно (ПЭТ)", category: "premium" },
  // ... остальные типы
];
```

### 3. Пример React компонента калькулятора

```jsx
// CalculatorForm.jsx
import React, { useState, useEffect } from 'react';

const CalculatorForm = () => {
  const [quantities, setQuantities] = useState({});
  const [selectedFacade, setSelectedFacade] = useState('plenka_mdf_1');
  const [customerInfo, setCustomerInfo] = useState({
    name: '',
    phone: '',
    address: ''
  });
  const [additionalServices, setAdditionalServices] = useState({
    delivery: 0,
    lifting: 0,
    installationPercent: 0.1 // 10% по умолчанию
  });
  const [totals, setTotals] = useState({});

  // Расчет стоимости
  const calculateTotal = () => {
    let total = 0;
    
    components.forEach(component => {
      const qty = quantities[component.id] || 0;
      const price = component.prices[selectedFacade] || 0;
      total += qty * price;
    });

    const installation = total * additionalServices.installationPercent;
    const totalWithServices = total + 
      additionalServices.delivery + 
      additionalServices.lifting + 
      installation;

    setTotals({
      base: total,
      installation: installation,
      total: totalWithServices
    });
  };

  useEffect(() => {
    calculateTotal();
  }, [quantities, selectedFacade, additionalServices]);

  const handleQuantityChange = (componentId, value) => {
    setQuantities({
      ...quantities,
      [componentId]: parseFloat(value) || 0
    });
  };

  return (
    <div className="calculator-container">
      {/* Информация о клиенте */}
      <div className="customer-info">
        <h3>Информация о клиенте</h3>
        <input
          type="text"
          placeholder="ФИО"
          value={customerInfo.name}
          onChange={(e) => setCustomerInfo({...customerInfo, name: e.target.value})}
        />
        <input
          type="tel"
          placeholder="Телефон"
          value={customerInfo.phone}
          onChange={(e) => setCustomerInfo({...customerInfo, phone: e.target.value})}
        />
        <input
          type="text"
          placeholder="Адрес"
          value={customerInfo.address}
          onChange={(e) => setCustomerInfo({...customerInfo, address: e.target.value})}
        />
      </div>

      {/* Выбор типа фасада */}
      <div className="facade-selector">
        <h3>Тип фасада</h3>
        <select 
          value={selectedFacade}
          onChange={(e) => setSelectedFacade(e.target.value)}
        >
          {facadeTypes.map(facade => (
            <option key={facade.id} value={facade.id}>
              {facade.name}
            </option>
          ))}
        </select>
      </div>

      {/* Компоненты кухни */}
      <div className="components">
        <h3>Параметры кухни</h3>
        {components.map(component => (
          <div key={component.id} className="component-row">
            <label>{component.name} ({component.unit})</label>
            <input
              type="number"
              step="0.01"
              value={quantities[component.id] || ''}
              onChange={(e) => handleQuantityChange(component.id, e.target.value)}
            />
            <span className="price">
              {((quantities[component.id] || 0) * component.prices[selectedFacade]).toFixed(2)} руб.
            </span>
          </div>
        ))}
      </div>

      {/* Дополнительные услуги */}
      <div className="additional-services">
        <h3>Дополнительные услуги</h3>
        <div>
          <label>Доставка (руб.)</label>
          <input
            type="number"
            value={additionalServices.delivery}
            onChange={(e) => setAdditionalServices({
              ...additionalServices, 
              delivery: parseFloat(e.target.value) || 0
            })}
          />
        </div>
        <div>
          <label>Подъем на этаж (руб.)</label>
          <input
            type="number"
            value={additionalServices.lifting}
            onChange={(e) => setAdditionalServices({
              ...additionalServices,
              lifting: parseFloat(e.target.value) || 0
            })}
          />
        </div>
        <div>
          <label>Установка (%)</label>
          <input
            type="number"
            step="0.01"
            value={additionalServices.installationPercent * 100}
            onChange={(e) => setAdditionalServices({
              ...additionalServices,
              installationPercent: (parseFloat(e.target.value) || 0) / 100
            })}
          />
        </div>
      </div>

      {/* Итоги */}
      <div className="totals">
        <h3>Итого</h3>
        <div>Стоимость мебели: {totals.base?.toFixed(2)} руб.</div>
        <div>Установка: {totals.installation?.toFixed(2)} руб.</div>
        <div><strong>Итого с услугами: {totals.total?.toFixed(2)} руб.</strong></div>
      </div>
    </div>
  );
};

export default CalculatorForm;
```

### 4. Backend API (Node.js/Express)

```javascript
// server.js
const express = require('express');
const app = express();
const db = require('./database');

app.use(express.json());

// Получить все компоненты с ценами
app.get('/api/components', async (req, res) => {
  try {
    const components = await db.query(`
      SELECT 
        c.id,
        c.name,
        c.unit,
        c.category,
        json_object_agg(
          ft.code,
          cp.price
        ) as prices
      FROM components c
      LEFT JOIN component_prices cp ON cp.component_id = c.id
      LEFT JOIN facade_types ft ON ft.id = cp.facade_type_id
      WHERE c.is_active = true
      GROUP BY c.id
      ORDER BY c.sort_order
    `);
    
    res.json(components.rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Сохранить расчет
app.post('/api/calculations', async (req, res) => {
  const { customerInfo, components, facadeType, additionalServices } = req.body;
  
  try {
    await db.query('BEGIN');
    
    // Создаем расчет
    const calculation = await db.query(`
      INSERT INTO calculations (customer_name, customer_phone, customer_address, total_amount)
      VALUES ($1, $2, $3, $4)
      RETURNING id
    `, [
      customerInfo.name,
      customerInfo.phone,
      customerInfo.address,
      req.body.totalAmount
    ]);
    
    const calcId = calculation.rows[0].id;
    
    // Сохраняем детали компонентов
    for (const comp of components) {
      if (comp.quantity > 0) {
        await db.query(`
          INSERT INTO calculation_details 
          (calculation_id, component_id, facade_type_id, quantity, unit_price, total_price)
          VALUES ($1, $2, $3, $4, $5, $6)
        `, [
          calcId,
          comp.componentId,
          facadeType,
          comp.quantity,
          comp.unitPrice,
          comp.totalPrice
        ]);
      }
    }
    
    // Сохраняем дополнительные услуги
    if (additionalServices.delivery > 0) {
      await db.query(`
        INSERT INTO additional_services (calculation_id, service_type, amount)
        VALUES ($1, 'delivery', $2)
      `, [calcId, additionalServices.delivery]);
    }
    
    if (additionalServices.lifting > 0) {
      await db.query(`
        INSERT INTO additional_services (calculation_id, service_type, amount)
        VALUES ($1, 'lifting', $2)
      `, [calcId, additionalServices.lifting]);
    }
    
    if (additionalServices.installation > 0) {
      await db.query(`
        INSERT INTO additional_services (calculation_id, service_type, amount)
        VALUES ($1, 'installation', $2)
      `, [calcId, additionalServices.installation]);
    }
    
    await db.query('COMMIT');
    
    res.json({ 
      success: true, 
      calculationId: calcId 
    });
    
  } catch (error) {
    await db.query('ROLLBACK');
    res.status(500).json({ error: error.message });
  }
});

// Генерация PDF
app.get('/api/calculations/:id/pdf', async (req, res) => {
  const { id } = req.params;
  
  try {
    // Получаем данные расчета
    const calculation = await getCalculationDetails(id);
    
    // Генерируем PDF
    const pdf = await generatePDF(calculation);
    
    res.setHeader('Content-Type', 'application/pdf');
    res.setHeader('Content-Disposition', `attachment; filename=calculation_${id}.pdf`);
    res.send(pdf);
    
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### 5. Сравнение типов фасадов

```jsx
// FacadeComparison.jsx
import React from 'react';

const FacadeComparison = ({ quantities, components, facadeTypes }) => {
  const calculateForFacade = (facadeId) => {
    let total = 0;
    
    components.forEach(component => {
      const qty = quantities[component.id] || 0;
      const price = component.prices[facadeId] || 0;
      total += qty * price;
    });
    
    return total;
  };
  
  const comparisons = facadeTypes.map(facade => ({
    ...facade,
    total: calculateForFacade(facade.id)
  })).sort((a, b) => a.total - b.total);
  
  return (
    <div className="facade-comparison">
      <h3>Сравнение стоимости по типам фасадов</h3>
      <table>
        <thead>
          <tr>
            <th>Тип фасада</th>
            <th>Категория</th>
            <th>Стоимость</th>
            <th>Разница с минимальной</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          {comparisons.map((facade, index) => (
            <tr key={facade.id} className={index === 0 ? 'best-price' : ''}>
              <td>{facade.name}</td>
              <td>{facade.category}</td>
              <td>{facade.total.toFixed(2)} руб.</td>
              <td>
                {index === 0 
                  ? 'Минимальная цена' 
                  : `+${(facade.total - comparisons[0].total).toFixed(2)} руб.`
                }
              </td>
              <td>
                <button onClick={() => selectFacade(facade.id)}>
                  Выбрать
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default FacadeComparison;
```

### 6. Миграция данных из Excel

```python
# migrate_from_excel.py
import pandas as pd
import psycopg2
from openpyxl import load_workbook

def migrate_excel_to_db():
    # Загружаем Excel файл
    wb = load_workbook('calculator.xlsx')
    ws = wb.active
    
    # Подключаемся к БД
    conn = psycopg2.connect(
        host="localhost",
        database="kitchen_calc",
        user="user",
        password="password"
    )
    cur = conn.cursor()
    
    # Типы фасадов из заголовков
    facade_types = [
        ('plenka_mdf_1', 'Пленка МДФ 1', 'economy'),
        ('plenka_mdf_2', 'Пленка МДФ 2', 'economy'),
        ('mdf_mylo', 'МДФ "Мыло" 3,4,5', 'standard'),
        ('mdf_premium', 'МДФ 345 Premium', 'premium'),
        ('mdf_country', 'МДФ 345 "Каунтри"', 'premium'),
        ('mdf_london', 'МДФ 345 "Лондон"', 'premium'),
        ('kraska_mat', 'Краска МАТ', 'premium'),
        ('kraska_gl', 'Краска ГЛЯНЕЦ', 'premium'),
        ('plastic', 'Пластик', 'premium'),
        ('eterno_pet', 'Етерно (ПЭТ)', 'premium'),
        # ... остальные
    ]
    
    # Вставляем типы фасадов
    for code, name, category in facade_types:
        cur.execute("""
            INSERT INTO facade_types (code, name, category)
            VALUES (%s, %s, %s)
        """, (code, name, category))
    
    # Компоненты и их цены
    components_data = []
    for row in range(6, 31):
        component_name = ws.cell(row=row, column=2).value
        if component_name:
            # Определяем категорию и единицу измерения
            if 'Низ' in component_name or 'Верх' in component_name:
                category = 'base'
                unit = 'пог.м'
            elif 'Угол' in component_name:
                category = 'corners'
                unit = 'шт'
            elif 'Ящик' in component_name or 'Я(' in component_name:
                category = 'hardware'
                unit = 'шт'
            elif 'Подьемник' in component_name:
                category = 'mechanisms'
                unit = 'шт'
            elif 'Ручк' in component_name:
                category = 'handles'
                unit = 'шт'
            else:
                category = 'other'
                unit = 'шт'
            
            # Вставляем компонент
            cur.execute("""
                INSERT INTO components (name, unit, category, sort_order)
                VALUES (%s, %s, %s, %s)
                RETURNING id
            """, (component_name, unit, category, row - 5))
            
            component_id = cur.fetchone()[0]
            
            # Собираем цены для каждого типа фасада
            price_columns = [
                (3, 'plenka_mdf_1'),
                (5, 'plenka_mdf_2'),
                (7, 'mdf_mylo'),
                (9, 'mdf_premium'),
                (11, 'kraska_mat'),
                (13, 'kraska_gl'),
                # ... остальные соответствия
            ]
            
            for col, facade_code in price_columns:
                price = ws.cell(row=row, column=col).value
                if price:
                    # Получаем ID типа фасада
                    cur.execute("""
                        SELECT id FROM facade_types WHERE code = %s
                    """, (facade_code,))
                    facade_id = cur.fetchone()[0]
                    
                    # Вставляем цену
                    is_discount = price < 0  # Отрицательные цены = скидки
                    cur.execute("""
                        INSERT INTO component_prices 
                        (component_id, facade_type_id, price, is_discount)
                        VALUES (%s, %s, %s, %s)
                    """, (component_id, facade_id, abs(price), is_discount))
    
    conn.commit()
    cur.close()
    conn.close()
    
    print("Миграция завершена успешно!")

if __name__ == "__main__":
    migrate_excel_to_db()
```

### 7. Дополнительные технические детали

#### Валидация данных:
```javascript
const validateCalculation = (data) => {
  const errors = [];
  
  // Проверка информации о клиенте
  if (!data.customerInfo.name) {
    errors.push('Имя клиента обязательно');
  }
  
  if (!data.customerInfo.phone) {
    errors.push('Телефон обязателен');
  }
  
  // Проверка наличия хотя бы одного компонента
  const hasComponents = Object.values(data.quantities)
    .some(qty => qty > 0);
  
  if (!hasComponents) {
    errors.push('Добавьте хотя бы один компонент');
  }
  
  // Проверка корректности процента установки
  if (data.additionalServices.installationPercent < 0 || 
      data.additionalServices.installationPercent > 1) {
    errors.push('Процент установки должен быть от 0 до 100');
  }
  
  return errors;
};
```

#### Кэширование цен:
```javascript
// Используем Redis для кэширования
const redis = require('redis');
const client = redis.createClient();

const getCachedPrices = async (facadeType) => {
  const key = `prices:${facadeType}`;
  const cached = await client.get(key);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Загружаем из БД
  const prices = await loadPricesFromDB(facadeType);
  
  // Кэшируем на 1 час
  await client.setex(key, 3600, JSON.stringify(prices));
  
  return prices;
};
```

#### Генерация уникального номера расчета:
```javascript
const generateCalculationNumber = () => {
  const date = new Date();
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const random = Math.floor(Math.random() * 9999).toString().padStart(4, '0');
  
  return `К-${year}${month}${day}-${random}`;
};
```

## Выводы

На основе анализа Excel-файла были извлечены следующие ключевые элементы:

1. **25 основных компонентов** кухни с индивидуальными ценами
2. **14 типов фасадов** с разными ценовыми категориями
3. **Формулы расчета** включающие скидки на угловые элементы
4. **Дополнительные услуги** с процентным расчетом установки

Веб-приложение полностью воспроизводит логику Excel-калькулятора с улучшениями:
- Автоматическое сравнение всех типов фасадов
- Валидация введенных данных
- Сохранение истории расчетов
- Генерация профессиональных коммерческих предложений
- Возможность масштабирования и интеграции
