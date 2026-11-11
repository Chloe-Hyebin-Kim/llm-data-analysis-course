# 6장. 데이터를 보며 질문을 만드는 EDA

이 노트북은 `book/chapters/ch06_eda_questions.md` 강의안을 초보자가 그대로 따라 하며 이해할 수 있도록 구성한 실습 자료입니다.

이번 장의 핵심은 그래프나 표를 많이 만드는 것이 아니라, 데이터를 보며 **현재 데이터로 답할 수 있는 질문**을 만들고 그 질문을 지표와 코드로 연결하는 것입니다.

## 0. 이 노트북 사용 방법

아래 셀을 위에서부터 차례대로 실행하세요.

- 이 노트북은 5장에서 만든 `data/processed/*_clean.csv` 파일을 사용합니다.
- 전처리 파일이 없다면 먼저 `python scripts/preprocess_data.py`를 실행하세요.
- EDA 결과표는 `reports/` 폴더에 CSV와 Markdown 보고서로 저장합니다.
- EDA 결과는 최종 결론이 아니라, 다음 분석 질문을 만드는 중간 산출물입니다.

## 1. EDA는 결론보다 질문에 가깝다

EDA는 Exploratory Data Analysis, 즉 탐색적 데이터 분석입니다. 이름 그대로 데이터를 여러 방향에서 탐색하면서 분포, 패턴, 차이, 관계, 이상한 값을 발견하는 과정입니다.

EDA에서 자주 던지는 질문은 다음과 같습니다.

- 데이터는 어떤 변수들로 구성되어 있는가?
- 주요 숫자형 변수의 분포는 어떤가?
- 주요 범주형 변수의 빈도는 어떤가?
- 특정 그룹 간 차이가 있는가?
- 시간에 따른 변화가 있는가?
- 이상하게 큰 값이나 작은 값이 있는가?
- 다음 단계에서 더 깊이 볼 질문은 무엇인가?

## 2. 관찰, 가설, 결론 구분하기

EDA 결과를 해석할 때는 관찰, 가설, 결론을 구분해야 합니다.

| 구분 | 의미 | 예시 |
|---|---|---|
| 관찰 | 데이터에서 직접 확인한 사실 | 전자기기 카테고리의 매출 비중이 가장 높다 |
| 가설 | 관찰을 바탕으로 생각해 볼 가능성 | 전자기기는 단가가 높아 매출 비중이 클 수 있다 |
| 결론 | 추가 검증 후 말할 수 있는 판단 | 단가와 판매 수량을 함께 분석한 결과 매출 차이의 주요 요인은 단가였다 |

이번 장에서는 주로 관찰과 가설을 만들고, 다음 분석 질문으로 연결하는 연습을 합니다.

## 3. 좋은 분석 질문의 조건

좋은 분석 질문은 데이터로 답할 수 있어야 합니다.

| 조건 | 설명 | 예시 |
|---|---|---|
| 데이터 기반 | 현재 데이터로 답할 수 있어야 함 | 월별 매출은 어떻게 변하는가? |
| 구체적 | 분석 대상과 기준이 명확해야 함 | 카테고리별 매출 비중은 어떻게 다른가? |
| 측정 가능 | 지표로 계산할 수 있어야 함 | 주문 수, 총매출, 평균 구매 금액 |
| 해석 가능 | 결과가 다음 판단과 연결되어야 함 | 어떤 상품군을 더 자세히 봐야 하는가? |
| 검증 가능 | 코드로 확인할 수 있어야 함 | `groupby()`로 계산 가능 |

## 4. 패키지와 경로 설정

노트북이 `notebooks/` 폴더 안에서 실행되는 경우와 프로젝트 루트에서 실행되는 경우를 모두 고려해 경로를 설정합니다.

from pathlib import Path
import sys

import pandas as pd
import numpy as np

CURRENT_DIR = Path.cwd()

#if CURRENT_DIR.name == 'notebooks':
if CURRENT_DIR.name == 'chapter06':  #내 과제에서는 llm-data-analysis-course\practice\chapter05 폴더 안에서 실행
    PROJECT_ROOT = CURRENT_DIR.parent.parent # 2단계 상위 폴더로 이동하여 프로젝트 루트 경로를 설정
else:
    PROJECT_ROOT = CURRENT_DIR

PROCESSED_DIR = PROJECT_ROOT / 'data' / 'processed'
REPORT_DIR = PROJECT_ROOT / 'reports'
REPORT_DIR.mkdir(exist_ok=True)

if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

print('현재 실행 위치:', CURRENT_DIR)
print('프로젝트 루트:', PROJECT_ROOT)
print('전처리 데이터 폴더:', PROCESSED_DIR)
print('보고서 폴더:', REPORT_DIR)

## 5. 전처리된 데이터 불러오기

6장은 5장에서 저장한 전처리 데이터를 사용합니다. 전처리 파일이 없다면 먼저 아래 명령을 터미널에서 실행하세요.

```bash
python scripts/preprocess_data.py
```

required_files = [
    PROCESSED_DIR / 'customers_clean.csv',
    PROCESSED_DIR / 'products_clean.csv',
    PROCESSED_DIR / 'orders_clean.csv',
    PROCESSED_DIR / 'order_items_clean.csv',
]

missing_files = [path for path in required_files if not path.exists()]

if missing_files:
    for path in missing_files:
        print('누락 파일:', path)
    raise FileNotFoundError('전처리 파일이 없습니다. 먼저 python scripts/preprocess_data.py 를 실행하세요.')

customers = pd.read_csv(PROCESSED_DIR / 'customers_clean.csv')
products = pd.read_csv(PROCESSED_DIR / 'products_clean.csv')
orders = pd.read_csv(PROCESSED_DIR / 'orders_clean.csv')
order_items = pd.read_csv(PROCESSED_DIR / 'order_items_clean.csv')

print('전처리 데이터 불러오기 완료')

## 6. 데이터 크기와 컬럼 확인하기

EDA를 시작하기 전에 각 데이터의 행/열 수와 컬럼명을 다시 확인합니다. CSV로 저장했다가 다시 불러오면 날짜 컬럼이 문자열로 돌아올 수 있으므로 날짜형 변환도 다시 확인합니다.

data = {
    'customers': customers,
    'products': products,
    'orders': orders,
    'order_items': order_items,
}

for name, df in data.items():
    print(f'[{name}]')
    print('shape:', df.shape)
    print('columns:', list(df.columns))
    print()

orders['order_date'] = pd.to_datetime(orders['order_date'], errors='coerce')

if 'signup_date' in customers.columns:
    customers['signup_date'] = pd.to_datetime(customers['signup_date'], errors='coerce')

if 'order_month' not in orders.columns:
    orders['order_month'] = orders['order_date'].dt.to_period('M').astype(str)

print('order_date 변환 실패:', orders['order_date'].isna().sum())
print('주문 시작일:', orders['order_date'].min())
print('주문 종료일:', orders['order_date'].max())

## 7. 분석 질문을 먼저 표로 정리하기

EDA를 시작하기 전에 어떤 질문을 볼지 먼저 표로 정리하면 분석이 산만해지지 않습니다. 아래 표는 고객, 상품, 주문, 매출, 시간, 고객 가치 관점에서 만들 수 있는 기본 질문입니다.

questions = pd.DataFrame({
    'analysis_area': ['고객', '상품', '주문', '매출', '시간', '고객 가치'],
    'question': [
        '고객은 어떤 지역과 연령대에 분포하는가?',
        '어떤 카테고리의 상품이 많은가?',
        '주문 상태와 결제수단 분포는 어떤가?',
        '카테고리별 매출은 어떻게 다른가?',
        '월별 매출과 주문 수는 어떻게 변하는가?',
        '구매 금액이 높은 고객은 누구인가?',
    ],
    'metric': [
        '고객 수, 평균 나이',
        '카테고리별 상품 수',
        '주문 수, 비율',
        '총매출, 매출 비중',
        '월별 매출, 월별 주문 수',
        '고객별 총매출, 주문 횟수',
    ],
    'required_data': [
        'customers',
        'products',
        'orders',
        'order_items, products',
        'orders, order_items',
        'customers, orders, order_items',
    ],
})

questions

## 8. 고객 데이터 EDA

먼저 고객 데이터만 따로 살펴봅니다. 하나의 데이터셋 안에서 분포와 빈도를 확인하면 이후 병합 분석을 할 때 기준이 생깁니다.

customers['age'].describe()

customer_city = customers['city'].value_counts(dropna=False).reset_index()
customer_city.columns = ['city', 'customer_count']
customer_city['customer_ratio'] = (
    customer_city['customer_count'] / customer_city['customer_count'].sum() * 100
).round(2)

customer_city

customer_gender = customers['gender'].value_counts(dropna=False).reset_index()
customer_gender.columns = ['gender', 'customer_count']
customer_gender['customer_ratio'] = (
    customer_gender['customer_count'] / customer_gender['customer_count'].sum() * 100
).round(2)

customer_gender

고객 데이터에서 이어질 수 있는 질문은 다음과 같습니다.

- 고객은 어느 도시에 많이 분포하는가?
- 고객의 평균 나이는 어느 정도인가?
- 성별 고객 수는 어떻게 분포하는가?
- 도시별 구매 금액도 차이가 있는가?

## 9. 상품 데이터 EDA

상품 데이터에서는 카테고리와 가격을 먼저 봅니다. 상품 수가 많은 카테고리가 매출도 높은지는 아직 알 수 없습니다. 그것은 주문 상세와 연결한 뒤 확인해야 합니다.

product_category = products['category'].value_counts(dropna=False).reset_index()
product_category.columns = ['category', 'product_count']
product_category['product_ratio'] = (
    product_category['product_count'] / product_category['product_count'].sum() * 100
).round(2)

product_category

products['price'].describe()

category_price = (
    products
    .groupby('category', as_index=False)
    .agg(
        product_count=('product_id', 'count'),
        avg_price=('price', 'mean'),
        min_price=('price', 'min'),
        max_price=('price', 'max'),
    )
    .assign(avg_price=lambda df: df['avg_price'].round(0))
    .sort_values('avg_price', ascending=False)
)

category_price

## 10. 주문 데이터 EDA

주문 데이터에서는 주문 상태, 결제수단, 주문 기간을 확인합니다. 주문 상태 분포는 이후 매출 분석에서 완료 주문만 볼지, 전체 주문을 볼지 결정하는 기준이 됩니다.

order_status = orders['order_status'].value_counts(dropna=False).reset_index()
order_status.columns = ['order_status', 'order_count']
order_status['order_ratio'] = (
    order_status['order_count'] / order_status['order_count'].sum() * 100
).round(2)

order_status

payment_method = orders['payment_method'].value_counts(dropna=False).reset_index()
payment_method.columns = ['payment_method', 'order_count']
payment_method['order_ratio'] = (
    payment_method['order_count'] / payment_method['order_count'].sum() * 100
).round(2)

payment_method

print('주문 시작일:', orders['order_date'].min())
print('주문 종료일:', orders['order_date'].max())
print('주문 월 개수:', orders['order_month'].nunique())

## 11. 매출 분석을 위한 데이터 연결

카테고리별 매출을 계산하려면 주문 상세와 상품 정보를 `product_id` 기준으로 연결해야 합니다. 병합 후에는 행 수와 누락 여부를 반드시 확인합니다.

if 'line_total' not in order_items.columns:
    order_items['line_total'] = order_items['quantity'] * order_items['unit_price']

order_items[['quantity', 'unit_price', 'line_total']].head()

sales_items = order_items.merge(
    products,
    on='product_id',
    how='left',
)

print('병합 전 order_items:', order_items.shape)
print('병합 후 sales_items:', sales_items.shape)
print('상품명 누락:', sales_items['product_name'].isna().sum())
print('카테고리 누락:', sales_items['category'].isna().sum())

sales_items.head()

## 12. 카테고리별 매출 EDA

카테고리별 매출은 어떤 상품군이 매출에 많이 기여했는지 보여 줍니다. 다만 매출이 높은 이유가 판매 수량 때문인지, 단가 때문인지는 추가로 확인해야 합니다.

category_sales = (
    sales_items
    .groupby('category', as_index=False)
    .agg(
        total_quantity=('quantity', 'sum'),
        total_sales=('line_total', 'sum'),
    )
    .sort_values('total_sales', ascending=False)
)

category_sales['sales_ratio'] = (
    category_sales['total_sales'] / category_sales['total_sales'].sum() * 100
).round(2)

category_sales

category_sales_with_price = category_sales.merge(
    category_price[['category', 'avg_price']],
    on='category',
    how='left',
)

category_sales_with_price

해석 예시:

> 관찰: 매출 비중이 높은 카테고리가 존재합니다.
> 가설: 해당 카테고리는 판매 수량이 많거나 평균 단가가 높아서 매출이 클 수 있습니다.
> 추가 분석: 카테고리별 판매 수량과 평균 단가를 함께 비교해야 합니다.

## 13. 월별 매출과 주문 수 EDA

월별 매출을 보려면 주문 상세와 주문 정보를 `order_id` 기준으로 연결합니다. 월별 매출은 시간 흐름을 보여 주지만, 원인을 바로 설명하지는 않습니다.

order_sales = order_items.merge(
    orders,
    on='order_id',
    how='left',
)

order_sales['order_date'] = pd.to_datetime(order_sales['order_date'], errors='coerce')
order_sales['order_month'] = order_sales['order_date'].dt.to_period('M').astype(str)

print('병합 후 order_sales:', order_sales.shape)
print('order_date 누락:', order_sales['order_date'].isna().sum())

order_sales.head()

monthly_sales = (
    order_sales
    .groupby('order_month', as_index=False)
    .agg(
        total_sales=('line_total', 'sum'),
        order_count=('order_id', 'nunique'),
    )
    .sort_values('order_month')
)

monthly_sales['avg_order_value'] = (
    monthly_sales['total_sales'] / monthly_sales['order_count']
).round(0)

monthly_sales

월별 매출이 특정 월에 높아졌다면 바로 원인을 단정하지 말고, 주문 수가 늘었는지 평균 주문 금액이 늘었는지, 특정 카테고리 매출이 늘었는지 추가로 확인해야 합니다.

## 14. 고객별 구매 금액 EDA

고객별 구매 금액은 우수 고객 분석의 출발점입니다. 하지만 한 번의 고액 구매 고객과 여러 번 반복 구매한 고객은 다르게 해석해야 합니다. 그래서 `total_sales`, `order_count`, `avg_order_value`를 함께 봅니다.

customer_sales_base = order_sales.merge(
    customers,
    on='customer_id',
    how='left',
)

group_columns = ['customer_id', 'city']
if 'name' in customer_sales_base.columns:
    group_columns = ['customer_id', 'name', 'city']

customer_sales = (
    customer_sales_base
    .groupby(group_columns, as_index=False)
    .agg(
        order_count=('order_id', 'nunique'),
        total_sales=('line_total', 'sum'),
    )
    .sort_values('total_sales', ascending=False)
)

customer_sales['avg_order_value'] = (
    customer_sales['total_sales'] / customer_sales['order_count']
).round(0)

customer_sales.head(10)

## 15. EDA 결과를 다음 질문으로 연결하기

EDA의 가치는 표를 만드는 데서 끝나지 않습니다. 각 결과가 다음 질문으로 이어져야 합니다.

eda_result_summary = pd.DataFrame({
    'question': [
        '고객은 어느 도시에 많이 분포하는가?',
        '어떤 카테고리의 상품이 많은가?',
        '카테고리별 매출은 어떻게 다른가?',
        '월별 매출과 주문 수는 어떻게 변하는가?',
        '구매 금액이 높은 고객은 누구인가?',
    ],
    'result_table': [
        'customer_city',
        'product_category',
        'category_sales',
        'monthly_sales',
        'customer_sales',
    ],
    'next_question': [
        '도시별 구매 금액도 차이가 있는가?',
        '상품 수가 많은 카테고리가 매출도 높은가?',
        '매출 차이가 수량 때문인가 단가 때문인가?',
        '특정 월의 매출 변화는 어떤 카테고리 때문인가?',
        '고액 구매 고객은 반복 구매 고객인가?',
    ],
})

eda_result_summary

## 16. EDA 결과 저장하기

EDA 결과표를 CSV로 저장하면 다음 장의 시각화나 보고서 작성에서 다시 사용할 수 있습니다.

customer_city.to_csv(REPORT_DIR / 'ch06_customer_city.csv', index=False, encoding='utf-8-sig')
product_category.to_csv(REPORT_DIR / 'ch06_product_category.csv', index=False, encoding='utf-8-sig')
category_sales.to_csv(REPORT_DIR / 'ch06_category_sales.csv', index=False, encoding='utf-8-sig')
monthly_sales.to_csv(REPORT_DIR / 'ch06_monthly_sales.csv', index=False, encoding='utf-8-sig')
customer_sales.to_csv(REPORT_DIR / 'ch06_customer_sales.csv', index=False, encoding='utf-8-sig')
eda_result_summary.to_csv(REPORT_DIR / 'ch06_eda_questions.csv', index=False, encoding='utf-8-sig')

list(REPORT_DIR.glob('ch06_*.csv'))

## 17. EDA 요약 보고서 만들기

간단한 Markdown 요약 보고서를 만들어 봅니다. 보고서에는 분석 목적, 주요 질문, 핵심 결과, 추가 질문, 해석 시 주의사항을 담습니다.

summary_text = f'''# Chapter 6 EDA 요약 보고서

## 1. 분석 목적

전처리된 온라인 쇼핑몰 데이터를 사용해 고객, 상품, 주문, 매출 관점의 기본 현황을 탐색했습니다.

## 2. 주요 분석 질문

```text
{questions.to_string(index=False)}
```

## 3. 카테고리별 매출 요약

```text
{category_sales.head(10).to_string(index=False)}
```

## 4. 월별 매출 요약

```text
{monthly_sales.to_string(index=False)}
```

## 5. 고객별 구매 금액 상위 10명

```text
{customer_sales.head(10).to_string(index=False)}
```

## 6. 추가 분석 질문

```text
{eda_result_summary.to_string(index=False)}
```

## 7. 해석 시 주의사항

- EDA 결과는 최종 결론이 아니라 추가 분석을 위한 관찰 결과입니다.
- 매출이 높은 카테고리가 반드시 선호도가 높은 카테고리라는 뜻은 아닙니다.
- 월별 매출 변화의 원인을 설명하려면 프로모션, 계절성, 신규 상품 등의 추가 정보가 필요합니다.
- 고객별 구매 금액은 주문 횟수와 평균 주문 금액을 함께 해석해야 합니다.
'''

report_path = REPORT_DIR / 'ch06_eda_summary.md'
report_path.write_text(summary_text, encoding='utf-8')

print('EDA 요약 보고서 저장 완료:', report_path)

## 18. 소스 모듈로 같은 EDA 실행하기

위에서 노트북으로 한 단계씩 실행한 EDA는 `src/eda.py`에 함수로 정리되어 있습니다. 반복 실행하거나 프로젝트 코드로 관리하려면 노트북보다 소스 모듈을 사용하는 것이 좋습니다.

from src.eda import (
    build_eda_report,
    load_processed_sales_data,
    run_basic_eda,
    save_eda_outputs,
)

module_data = load_processed_sales_data(PROCESSED_DIR)
module_results = run_basic_eda(module_data)

module_results.keys()

module_results['category_sales'].head()

## 19. 스크립트로 한 번에 실행하기

노트북에서 한 단계씩 이해한 EDA 과정을 스크립트로도 실행할 수 있습니다. 터미널에서 프로젝트 루트 기준으로 아래 명령을 실행합니다.

```bash
python scripts/run_eda.py
```

이 스크립트는 6장 EDA 결과 CSV와 `reports/ch06_eda_summary.md`를 자동으로 저장합니다.

# 개인 노트! - 프로젝트 경로 관련 트러블슈팅

파일 경로를 직접 지정해서 실행하면서 Python이 프로젝트 루트의 src 패키지를 찾지 못함.

해결-> ```python -m``` 을 사용!

Python에게 파일 경로로 스크립트를 실행하는 대신, 모듈 이름을 기준으로 찾아서 실행하라고 지시.


⭐핵심 정리⭐
- python script.py: 파일 경로를 지정해서 실행합니다.
- python -m package.module: 모듈 이름을 이용해 실행합니다.
- sys.path: Python이 모듈을 검색하는 경로 목록입니다.



![프로젝트 경로 관련 트러블슈팅](images/ModuleNotFoundError_troubleShooting.png)

## 20. LLM과 함께 질문을 확장하기

LLM은 EDA 질문을 확장하고 결과 해석 문장을 다듬는 데 도움이 됩니다. 하지만 LLM이 제안한 질문이 실제 데이터로 답할 수 있는지는 반드시 사람이 확인해야 합니다.

```text
온라인 쇼핑몰 데이터로 EDA를 수행하려고 합니다.

데이터셋:
- customers: customer_id, gender, age, city, signup_date
- products: product_id, product_name, category, price
- orders: order_id, customer_id, order_date, payment_method, order_status
- order_items: order_id, product_id, quantity, unit_price, line_total

요청:
1. 현재 데이터로 검증 가능한 EDA 질문 10개를 만들어 주세요.
2. 각 질문에 필요한 데이터셋과 pandas 기능을 함께 정리해 주세요.
3. 데이터에 없는 내용은 추측하지 마세요.
4. 각 질문을 계산 가능한 지표로 바꿔 주세요.
```

## 21. LLM 결과 해석 요청 예시

EDA 결과 해석을 요청할 때는 원인 단정을 막는 조건을 넣는 것이 좋습니다.

```text
다음은 온라인 쇼핑몰 카테고리별 매출 요약 결과입니다.

category,total_quantity,total_sales,sales_ratio
전자기기,320,12500000,42.5
생활용품,510,7800000,26.5
패션,260,6200000,21.1
식품,430,2900000,9.9

이 결과를 보고서에 넣을 수 있도록 해석해 주세요.

조건:
- 데이터에 없는 원인을 단정하지 말 것
- 원인 설명은 가설로 표현할 것
- 추가로 확인해야 할 분석 질문을 제안할 것
- 관찰, 가설, 추가 분석을 구분할 것
```

## 22. 실습 과제

아래 과제를 직접 해결해 보세요.

1. 도시별 고객 수와 도시별 총 구매 금액을 비교하세요.
2. 상품 수가 많은 카테고리가 매출도 높은지 확인하세요.
3. 결제수단별 평균 주문 금액을 계산하세요.
4. 월별·카테고리별 매출 요약표를 만들어 보세요.
5. 고객별 `order_count`와 `avg_order_value`를 함께 보고 반복 구매 고객과 고액 단발 구매 고객을 구분하는 기준을 생각해 보세요.
6. LLM에게 현재 데이터로 답할 수 없는 질문을 답할 수 있는 질문으로 바꾸게 하는 프롬프트를 작성해 보세요.

# 매출 분석에 필요한 준비

# 주문 상세의 상품별 금액
if "line_total" not in order_items.columns:
    order_items["line_total"] = (order_items["quantity"] * order_items["unit_price"])

# 완료된 주문만 매출 분석에 사용
sales_orders = orders[orders["order_status"] == "completed"].copy()

print("전체 주문 수:", len(orders))
print("완료 주문 수:", len(sales_orders))
print("주문 상세 행 수:", len(order_items))
# 과제 1. 도시별 고객 수와 도시별 총 구매 금액을 비교하세요.

# 1. 도시별 고객 수
city_customers = (customers.groupby("city", as_index=False)
                  .agg(customer_count=("customer_id", "nunique")))

# 2. 주문 상세 + 완료 주문 + 고객 정보 연결
city_sales_data = (order_items.merge(sales_orders[["order_id", "customer_id"]],on="order_id",how="inner")
    .merge(customers[["customer_id", "city"]],on="customer_id",how="left"))

# 3. 도시별 총 구매 금액
city_sales = (city_sales_data.groupby("city", as_index=False)
              .agg(total_purchase_amount=("line_total", "sum")))

# 4. 고객 수와 구매 금액 결합
city_summary = (city_customers.merge(city_sales, on="city", how="left").fillna({"total_purchase_amount": 0})
                .sort_values("total_purchase_amount", ascending=False))

city_summary["total_purchase_amount"] = (city_summary["total_purchase_amount"].round(0))

display(city_summary)
### [해석할 내용]
- 고객 수가 많으면서 구매 금액도 높은 도시가 있는가?
- 고객 수는 적지만 구매 금액이 높은 도시가 있는가?
- 도시별 구매 금액 차이가 고객 수 차이와 관련이 있어 보이는가?
city_summary["purchase_per_customer"] = (city_summary["total_purchase_amount"]/ city_summary["customer_count"]).round(0)

display(city_summary.sort_values("purchase_per_customer",ascending=False))
# 과제 2. 상품 수가 많은 카테고리가 매출도 높은지 확인하세요.

# 1. 카테고리별 등록 상품 수
category_product_count = (products.groupby("category", as_index=False)
                              .agg(product_count=("product_id", "nunique")))

# 2. 주문 상세와 상품 정보를 product_id로 연결
category_sales_data = (order_items.merge(sales_orders[["order_id"]],on="order_id",how="inner")
                       .merge(products[["product_id", "category"]],on="product_id",how="left"))

# 3. 카테고리별 판매량과 매출
category_revenue = (category_sales_data.groupby("category", as_index=False)
                    .agg(total_quantity=("quantity", "sum"),total_sales=("line_total", "sum")))

# 4. 상품 수와 매출 결합
category_summary = (category_product_count.merge(category_revenue, on="category", how="left")
                    .fillna({"total_quantity": 0,"total_sales": 0})
                    .sort_values("total_sales", ascending=False))

display(category_summary)
#상품 수와 매출의 상관계수
#양수: 상품 수가 많은 카테고리일수록 매출도 높은 경향
#음수: 상품 수가 많은 카테고리일수록 매출이 낮은 경향
#0에 가까움: 두 변수 사이에 뚜렷한 선형 관계가 약함

correlation = category_summary[["product_count", "total_sales"]].corr().iloc[0, 1]

print(f"상품 수와 매출의 상관계수: {correlation:.3f}")
#### 주의!!!
카테고리가 개수에 따라 상관계수 계산에 사용되는 관측치도 영향을 받기때문에  결과는 탐색적 참고 지표로 사용해야 하며, 상품 수가 매출을 높인다는 인과관계를 의미하지 않음!
# 과제 3. 결제수단별 평균 주문 금액을 계산하세요.

# 1. 주문별 총금액 계산
order_amounts = (order_items.merge(sales_orders[["order_id", "payment_method"]],on="order_id",how="inner")
                 .groupby(["order_id", "payment_method"],as_index=False)
                 .agg(order_amount=("line_total", "sum")))

# 2. 결제수단별 평균 주문 금액
payment_summary = (order_amounts.groupby("payment_method", as_index=False)
                   .agg(order_count=("order_id", "nunique"),avg_order_amount=("order_amount", "mean"),total_sales=("order_amount", "sum"))
                   .sort_values("avg_order_amount", ascending=False))

payment_summary["avg_order_amount"] = (payment_summary["avg_order_amount"].round(0))

display(payment_summary)

[해석할 내용]
- 평균 주문 금액이 가장 높은 결제수단은 무엇인가?
- 주문 수가 많은 결제수단과 평균 주문 금액이 높은 결제수단이 같은가?
- 특정 결제수단의 평균 주문 금액이 높더라도 주문 수가 적지는 않은가?

여기서 평균 주문 금액은 주문 하나의 총금액을 구한 뒤 결제수단별로 평균을 계산한 값! 결제수단별 평균 주문 금액이 다르다고 해서 결제수단 자체가 구매 금액을 높인다고 결론 내릴 수는 없다. 고객 특성이나 구매 상품 구성 등이 다를 수 있기 때문.
# 과제 4. 월별·카테고리별 매출 요약표를 만들어 보세요.

# 1. 주문 상세에 주문 날짜 연결
monthly_category_data = (order_items.merge(sales_orders[["order_id", "order_date"]],on="order_id",how="inner")
                         .merge(products[["product_id", "category"]],on="product_id",how="left"))

# 2. 날짜를 datetime 형식으로 변환
monthly_category_data["order_date"] = pd.to_datetime(monthly_category_data["order_date"],errors="coerce")

# 3. 연-월 컬럼 생성
monthly_category_data["order_month"] = (monthly_category_data["order_date"].dt.to_period("M").astype(str))

# 4. 월별·카테고리별 매출 집계
monthly_category_sales = (monthly_category_data.groupby(["order_month", "category"],as_index=False)
                          .agg(total_sales=("line_total", "sum"),total_quantity=("quantity", "sum"))
                          .sort_values(["order_month", "category"]))

display(monthly_category_sales)
# 행: 월 / 열: 카테고리 / 값: 매출

monthly_category_pivot = pd.pivot_table(
    monthly_category_sales,
    index="order_month",
    columns="category",
    values="total_sales",
    aggfunc="sum",
    fill_value=0
)

display(monthly_category_pivot)
- 특정 월에 매출이 급증한 카테고리가 있는가?
- 매출이 꾸준한 카테고리와 변동이 큰 카테고리는 무엇인가?
- 전체 매출 변화가 특정 카테고리에 집중되어 있는가?
#분석 결과를 CSV로 저장하기
from pathlib import Path

REPORT_DIR = Path("reports")
REPORT_DIR.mkdir(parents=True, exist_ok=True)

city_summary.to_csv(
    REPORT_DIR / "ch06_exercise_city_sales.csv",
    index=False,
    encoding="utf-8-sig"
)

category_summary.to_csv(
    REPORT_DIR / "ch06_exercise_category_sales.csv",
    index=False,
    encoding="utf-8-sig"
)

payment_summary.to_csv(
    REPORT_DIR / "ch06_exercise_payment_summary.csv",
    index=False,
    encoding="utf-8-sig"
)

monthly_category_sales.to_csv(
    REPORT_DIR / "ch06_exercise_monthly_category_sales.csv",
    index=False,
    encoding="utf-8-sig"
)

monthly_category_pivot.to_csv(
    REPORT_DIR / "ch06_exercise_monthly_category_pivot.csv",
    encoding="utf-8-sig"
)

print("Chapter 06 실습 결과 저장 완료")
# 5. 고객별 order_count와 avg_order_value를 함께 보고 반복 구매 고객과 고액 단발 구매 고객을 구분하는 기준 생각해보기

[분석 목적과 지표 정의]
- order_count: 고객별 주문 건수
- avg_order_value: 고객별 평균 주문 금액. 고객의 전체 주문 금액을 주문 건수로 나눈 값
    
두 지표를 함께 보면 구매 빈도와 주문당 금액을 구분해서 살펴볼 수 있다.

# 주문 상품 금액을 합산해 주문별 금액을 먼저 계산한 뒤, 고객별 지표를 집계
# 해당 코드는 orders에 포함된 모든 주문 상태를 집계하기떄문에, 취소 및 환불 주문을 제외하려면 분석 목적에 맞는 상태 기준을 먼저 정해야 함.
import pandas as pd

# 1. 주문별 총금액 계산
order_amount = (
    order_items
    .groupby("order_id", as_index=False)
    .agg(order_amount=("line_total", "sum"))
)

# 2. 주문 정보와 주문별 금액 결합
order_detail = orders.merge(
    order_amount,
    on="order_id",
    how="left",
    validate="one_to_one"
)

# 3. 고객별 주문 건수와 평균 주문 금액
customer_summary = (
    order_detail
    .groupby("customer_id", as_index=False)
    .agg(
        order_count=("order_id", "nunique"),
        avg_order_value=("order_amount", "mean"),
        total_order_amount=("order_amount", "sum")
    )
)

# 4. 주문 건수가 많은 고객부터 확인
customer_summary = customer_summary.sort_values(
    ["order_count", "avg_order_value"],
    ascending=[False, False]
)

display(customer_summary.head(10))

display(
    customer_summary[
        ["order_count", "avg_order_value", "total_order_amount"]
    ].describe()
)

display(
    customer_summary.sort_values(
        "avg_order_value", ascending=False
    ).head(10)
)
# 6. 답할 수 없는 질문을 답할 수 있는 질문으로 바꾸는 프롬프트

```text
너는 데이터 분석 설계 전문가다.

## 목표
현재 보유한 데이터로 답할 수 없는 질문을 식별하고, 실제 데이터로 검증할 수 있는 질문으로 바꿔라.

## 데이터 스키마
- customers: `customer_id`, `gender`, `age`, `city`, `signup_date`
- products: `product_id`, `product_name`, `category`, `price`
- orders: `order_id`, `customer_id`, `order_date`, `payment_method`, `order_status`
- order_items: `order_id`, `product_id`, `quantity`, `unit_price`, `line_total`

## 작업 지침
1. 원래 질문에 답하기 위해 필요한 정보와 지표를 파악한다.
2. 현재 스키마에 해당 정보가 존재하는지 확인한다.
3. 데이터에 없는 정보가 필요하면 그 정보가 없다는 점을 명시한다.
4. 현재 데이터로 직접 계산할 수 있는 지표를 사용해 대체 질문을 작성한다.
5. 대체 질문에 사용할 데이터프레임, 컬럼, 집계 방법을 제시한다.
6. 필요한 pandas 함수와 계산식을 제안한다.
7. 결과를 해석할 때 관찰된 사실과 추정·가설을 구분한다.
8. 데이터만으로 확인할 수 없는 인과관계나 고객의 심리적 이유는 단정하지 않는다.

## 출력 형식
- 원래 질문:
- 현재 데이터만으로 답할 수 있는가: 가능 / 일부 가능 / 불가능
- 부족한 데이터:
- 검증 가능한 대체 질문:
- 사용할 데이터와 컬럼:
- 계산할 지표:
- 분석 방법:
- 해석 시 주의점:

## 변환 예시
원래 질문: “고객이 왜 우리 제품을 구매했는가?”

문제점: 현재 데이터에는 구매 동기, 설문 응답, 고객 인터뷰 내용이 없다.

대체 질문: “상품 카테고리별 판매 수량과 매출은 어떻게 다른가?”

사용할 데이터와 컬럼: `products.category`, `order_items.quantity`, `order_items.line_total`

계산 지표: 카테고리별 총판매 수량, 총매출, 매출 비중

분석 방법: 상품 정보와 주문 상품 정보를 결합하고 카테고리별로 집계한다.

이제 다음 질문을 위 형식에 맞춰 분석하라.


요청:
1. 현재 데이터로 검증 가능한 EDA 질문 10개를 만들어 주세요.
2. 각 질문에 필요한 데이터셋과 pandas 기능을 함께 정리해 주세요.
3. 데이터에 없는 내용은 추측하지 마세요.
4. 각 질문을 계산 가능한 지표로 바꿔 주세요.

```
## 23. 정리

이번 장에서는 다음 내용을 실습했습니다.

- EDA에서 관찰, 가설, 결론 구분하기
- 현재 데이터로 답할 수 있는 질문 만들기
- 고객, 상품, 주문 데이터의 기본 분포 확인
- 주문 상세, 상품, 주문, 고객 데이터를 병합해 매출 분석하기
- 카테고리별 매출, 월별 매출, 고객별 구매 금액 계산하기
- EDA 결과를 다음 질문으로 연결하기
- EDA 결과 CSV와 Markdown 보고서 저장하기
- `src/eda.py`와 `scripts/run_eda.py`로 반복 실행 가능한 소스 구조 만들기

다음 장에서는 EDA에서 만든 질문과 집계표를 바탕으로 데이터 시각화를 다룹니다.


## 1. EDA 질문 10개와 계산 지표

현재 확인 가능한 컬럼만 사용했습니다. `line_total`은 제공된 주문 상품 데이터의 금액 컬럼입니다.

1\. 도시별 고객 수와 매출은 어떻게 다른가?

분석 목적: 고객 분포와 매출 규모의 지역별 차이 파악

- 계산 지표: 도시별 고객 수, 총매출, 고객 1인당 매출
- 필요 데이터: `customers`, `orders`, `order_items`
- 계산 방법: 고객 수는 도시별 고유 `customer_id` 수, 매출은 주문과 주문 상품을 연결한 `line_total` 합계
- pandas: `merge()`, `groupby()`, `nunique()`, `sum()`, `fillna()`

2\. 상품 수가 많은 카테고리는 매출도 높은가?

분석 목적: 상품 구색 규모와 실제 매출 사이의 관계 탐색

- 계산 지표: 카테고리별 상품 수, 총매출, 상품 수와 총매출의 상관계수
- 필요 데이터: `products`, `order_items`, `orders`
- 계산 방법: `products`에서 카테고리별 고유 상품 수를 세고, 주문 상품을 카테고리와 연결해 매출 합계 계산
- pandas: `groupby()`, `nunique()`, `merge()`, `sum()`, `corr()`

상관관계는 연관성을 나타낼 뿐, 상품 수가 매출을 증가시킨다는 인과관계를 입증하지 않습니다.

3\. 결제 수단별 평균 주문 금액은 어떻게 다른가?

분석 목적: 결제 수단별 주문 금액 특성 비교

- 계산 지표: 결제 수단별 주문 수, 평균 주문 금액, 총매출
- 필요 데이터: `orders`, `order_items`
- 계산 방법: `order_id`별 `line_total`을 먼저 합산한 뒤 `payment_method`별 평균 계산
- pandas: `merge()`, `groupby()`, `sum()`, `mean()`, `agg()`

주문별 합계를 먼저 계산해야 상품 행을 그대로 평균 내는 오류를 방지할 수 있습니다.

4\. 월별 매출은 어떤 추세를 보이는가?

분석 목적: 매출의 시간적 변화 확인

- 계산 지표: 월별 총매출, 월별 주문 수, 전월 대비 매출 증감률
- 필요 데이터: `orders`, `order_items`
- 계산 방법: `order_date`를 월 단위로 변환하고 월별 매출과 고유 주문 수를 집계
- pandas: `to_datetime()`, `dt.to_period()`, `groupby()`, `nunique()`, `pct_change()`

전월 대비 증감률은 비교할 이전 달의 매출이 있을 때 계산할 수 있습니다.

5\. 월별·카테고리별 매출 구성은 어떻게 변하는가?

분석 목적: 카테고리별 매출 변화와 월별 구성 비교

- 계산 지표: 월별·카테고리별 매출, 월별 카테고리 매출 비중(%)
- 필요 데이터: `orders`, `order_items`, `products`
- 계산 방법: 주문 상품을 날짜와 카테고리에 연결한 뒤 월·카테고리별 합계 및 해당 월 매출 대비 비율 계산
- pandas: `merge()`, `groupby()`, `pivot_table()`, `div()`, `unstack()`

6\. 매출이 가장 높은 상품과 가장 낮은 상품은 무엇인가?

분석 목적: 상품별 매출 기여도와 매출 분포 파악

- 계산 지표: 상품별 총매출, 판매 수량, 매출 순위
- 필요 데이터: `products`, `order_items`
- 계산 방법: `product_id`별 `line_total`과 `quantity`를 합산하고 내림차순·오름차순 정렬
- pandas: `groupby()`, `sum()`, `sort_values()`, `rank()`

판매 이력이 없는 상품은 주문 상품 데이터에 나타나지 않을 수 있으므로 전체 상품 목록과 결합해야 합니다.

7\. 연령대별 고객 수와 구매 실적은 어떻게 다른가?

분석 목적: 연령에 따른 고객 분포 및 구매 실적 비교

- 계산 지표: 연령대별 고객 수, 구매 고객 수, 총매출, 구매 고객당 매출
- 필요 데이터: `customers`, `orders`, `order_items`
- 계산 방법: `age`를 구간화하고 고객별 구매 실적을 연결해 연령대별 집계
- pandas: `cut()`, `merge()`, `groupby()`, `nunique()`, `sum()`

연령대 경계는 분석자가 정해야 하며, 연령이 비어 있는 고객은 별도 처리해야 합니다.

8\. 고객 중 반복 구매 고객의 비율은 얼마인가?

분석 목적: 관측된 주문 데이터에서 재구매 패턴 확인

- 계산 지표: 고객별 주문 수, 2회 이상 주문한 고객 비율(%), 고객별 평균 주문 수
- 필요 데이터: `customers`, `orders`
- 계산 방법: 고객별 고유 `order_id` 수를 계산하고 2회 이상 주문한 고객 비율 산출
- pandas: `groupby()`, `nunique()`, `mean()`, `value_counts()`

이는 데이터에 기록된 기간의 반복 주문 비율입니다. 실제 고객 유지율이나 장기 충성도를 판단하려면 관측 기간 정보가 필요합니다.

9\. 주문 상태별 주문 수와 매출은 어떻게 다른가?

분석 목적: 주문 상태에 따른 데이터 규모 및 금액 차이 확인

- 계산 지표: 상태별 주문 수, 상태별 주문 상품 금액 합계, 상태별 주문 비율(%)
- 필요 데이터: `orders`, `order_items`
- 계산 방법: 주문 상태별로 주문을 집계하고 주문 상품 금액을 연결해 합산
- pandas: `merge()`, `groupby()`, `nunique()`, `sum()`, `value_counts()`

상태별 금액 합계는 계산 가능하지만, 환불·취소 금액의 회계적 처리는 상태 정의와 업무 규칙을 확인해야 합니다.

10\. 가입 월별 고객의 구매 실적은 어떻게 다른가?

분석 목적: 가입 시점에 따른 관측 구매 실적 비교

- 계산 지표: 가입 월별 고객 수, 구매 고객 비율(%), 고객당 누적 매출
- 필요 데이터: `customers`, `orders`, `order_items`
- 계산 방법: `signup_date`를 월 단위로 묶고 고객별 주문·매출을 연결하여 가입 월별 집계
- pandas: `to_datetime()`, `dt.to_period()`, `merge()`, `groupby()`, `nunique()`, `sum()`

가입 월별 비교에는 관측 기간 차이가 영향을 줄 수 있습니다. 따라서 이 지표만으로 가입 시점이 구매를 유발했다고 결론 내릴 수 없습니다.

## 2. 분석 질문별 데이터셋 요약표

| 번호 | 분석 질문             | 필요한 데이터셋                             | 핵심 지표                     |
| -- | ----------------- | ------------------------------------ | ------------------------- |
| 1  | 도시별 고객 수와 매출      | `customers`, `orders`, `order_items` | 고객 수, 총매출, 고객당 매출         |
| 2  | 상품 수와 카테고리 매출의 관계 | `products`, `orders`, `order_items`  | 상품 수, 총매출, 상관계수           |
| 3  | 결제 수단별 평균 주문 금액   | `orders`, `order_items`              | 주문 수, 평균 주문 금액, 총매출       |
| 4  | 월별 매출 추세          | `orders`, `order_items`              | 월별 매출, 주문 수, 증감률          |
| 5  | 월별·카테고리별 매출 구성    | `orders`, `order_items`, `products`  | 카테고리 매출, 월별 매출 비중         |
| 6  | 상품별 매출 순위         | `products`, `order_items`            | 상품별 매출, 판매 수량, 순위         |
| 7  | 연령대별 구매 실적        | `customers`, `orders`, `order_items` | 고객 수, 구매 고객 비율, 매출        |
| 8  | 반복 구매 비율          | `customers`, `orders`                | 고객별 주문 수, 반복 구매 비율        |
| 9  | 주문 상태별 주문·매출      | `orders`, `order_items`              | 상태별 주문 수, 금액, 주문 비율       |
| 10 | 가입 월별 구매 실적       | `customers`, `orders`, `order_items` | 가입 고객 수, 구매 고객 비율, 고객당 매출 |

## 3. 분석 전에 확인할 사항

EDA 결과의 정확도를 높이려면 다음 항목을 먼저 점검하는 것이 좋습니다.

- 주문 상태 기준: 전체 주문을 포함할지, 완료된 주문만 매출로 볼지 정의합니다. 상태별 분석은 상태를 유지한 채 별도로 집계하는 것이 좋습니다.
- 집계 단위: 주문 수는 `order_id`의 고유 개수, 고객 수는 `customer_id`의 고유 개수로 계산합니다. 주문 상품 행 수를 주문 수로 간주하면 안 됩니다.
- 중복 조인 방지: `orders`와 `products`의 키가 유일한지 확인합니다. 조인 과정에서 행이 늘어나면 매출 합계가 부풀려질 수 있습니다.
- 금액 기준: 제공된 `line_total`을 매출 지표로 사용하되, 할인·환불·배송비가 포함되는지는 현재 스키마만으로 확인할 수 없습니다.
- 시간 기준: `order_date`와 `signup_date`의 자료형 및 결측치를 점검합니다. 가입 월별 실적은 고객별 관측 기간 차이도 고려해야 합니다.

제공된 집계표만으로 확인되는 사실은 스포츠 카테고리가 표에 제시된 카테고리 중 총매출과 판매 수량이 가장 높고, 패션 카테고리는 두 지표 모두 가장 낮다는 것입니다. 상품 수, 고객 특성, 결제 수단, 월별 추세와의 관계는 해당 원본 데이터로 추가 계산해야 검증할 수 있습니다.
# Chapter 06 답안. 데이터를 보며 질문을 만드는 EDA

## 0. 제출 정보
- 이름: 김혜빈
- GitHub ID: Chloe-Hyebin-Kim
- 작성일: 2026-10-21
- 최종 제출 URL: https://github.com/Chloe-Hyebin-Kim/llm-data-analysis-course/blob/main/practice/chapter06/chapter06.ipynb


# Chapter 06 답안: 데이터를 보며 질문을 만드는 EDA

## 1. 내가 정한 EDA 질문

| 질문 | 분석 범위 | 지표 | 검증 방법 |
| --- | --- | --- | --- |
| completed 주문에서 어떤 카테고리가 매출과 판매량을 주도하는가? | completed 주문 상세 + 상품 카테고리 (products에 없는 상품은 `Unknown`) | `line_total` 합계, 판매 수량, 매출 비중, 평균 판매 단가 | 카테고리별 합계(Unknown 포함)가 completed 원본 합계와 같은지 비교 |
| completed 매출은 월별로 어떻게 변하는가? | completed 주문 (주문일 2025-07-01~2026-08-30, 미래 날짜·날짜 누락 주문은 별도 표시) | 월별 매출, 월별 주문 수, 주문당 평균 금액 | 월별 합계 + 주문일 누락 금액 = completed 원본 합계인지 확인 |
| 결제수단별 completed 주문 금액은 차이가 있는가? | completed 주문을 결제수단별로 집계 | 주문 수, 총매출, 주문당 평균 금액 | 주문별 금액을 먼저 합산한 뒤 계산하고, 주문 수 합계(180건)와 총매출을 대조 |
 
### 왜 이 질문을 선택했는가?
세 질문 모두 현재 데이터의 주문 상태, 주문일, 결제수단, 상품 카테고리, 수량, `line_total`만으로 계산할 수 있습니다. 카테고리·시간·결제수단은 같은 completed 금액을 서로 다른 기준으로 나눈 것이므로, 세 집계의 합계를 원본 합계와 비교하면 병합 누락이나 결측 때문에 빠진 금액을 찾을 수 있습니다. 실제로 이 과정에서 카테고리 누락과 주문일 누락을 발견했습니다.
 
## 2. 핵심 집계 결과
- completed 원본 합계: **141,340,000원** (completed 주문 180건, 주문 상세 446행)
- 카테고리별 합계: **141,340,000원** (`Unknown` 포함 시 원본과 일치)

| 카테고리 | completed 매출 | 판매 수량 | 매출 비중 | 평균 판매 단가 |
| --- | ---: | ---: | ---: | ---: |
| 패션 | 32,940,000원 | 290 | 23.3% | 113,586원 |
| 식품 | 25,035,000원 | 226 | 17.7% | 110,774원 |
| 전자기기 | 21,485,000원 | 176 | 15.2% | 122,074원 |
| 도서 | 17,663,000원 | 174 | 12.5% | 101,511원 |
| 뷰티 | 17,103,000원 | 168 | 12.1% | 101,804원 |
| 스포츠 | 13,237,000원 | 165 | 9.4% | 80,224원 |
| 생활용품 | 7,001,000원 | 108 | 5.0% | 64,824원 |
| Unknown | 4,015,000원 | 34 | 2.8% | 118,088원 |
| 생활 용품 | 2,108,000원 | 17 | 1.5% | 124,000원 |
| 뷰 티 | 648,000원 | 12 | 0.5% | 54,000원 |
| 전자 기기 | 105,000원 | 5 | 0.1% | 21,000원 |
 
\* 매출 비중은 소수점 첫째 자리에서 반올림해 합계가 100.1%입니다. 카테고리 표기 차이(`생활 용품`, `뷰 티`, `전자 기기`)는 표준화하지 않고 관찰로 기록했습니다.
 
- 월별 합계: **139,437,000원** (원본보다 1,903,000원 적음)

| 월 | 매출 | 주문 수 | 주문당 평균 |
| --- | ---: | ---: | ---: |
| 2025-07 | 9,561,000원 | 11 | 869,182원 |
| 2025-08 | 9,362,000원 | 17 | 550,706원 |
| 2025-09 | 5,299,000원 | 4 | 1,324,750원 |
| 2025-10 | 12,629,000원 | 17 | 742,882원 |
| 2025-11 | 9,583,000원 | 16 | 598,938원 |
| 2025-12 | 8,581,000원 | 12 | 715,083원 |
| 2026-01 | 8,135,000원 | 9 | 903,889원 |
| 2026-02 | 7,689,000원 | 12 | 640,750원 |
| 2026-03 | 7,247,000원 | 13 | 557,462원 |
| 2026-04 | 11,459,000원 | 14 | 818,500원 |
| 2026-05 | 12,006,000원 | 12 | 1,000,500원 |
| 2026-06 | 11,031,000원 | 16 | 689,438원 |
| 2026-07 | 8,966,000원 | 8 | 1,120,750원 |
| 2026-08 | 16,980,000원 | 15 | 1,132,000원 |
| 2027-01\* | 909,000원 | 1 | 909,000원 |
| 주문일 누락 | 1,903,000원 | 2 | 951,500원 |
 
\* 2027-01은 미래 날짜(2027-01-15) 주문 1건입니다. 주문일 누락은 날짜 변환에 실패한 order 72, 214이며 월별 집계에서 빠집니다.
 
- 총합 일치 여부: **부분 일치**

| 집계 기준 | 합계 | 원본과 차이 | 차이 원인 |
| --- | ---: | ---: | --- |
| completed 원본 | 141,340,000원 | 0 | - |
| 카테고리별 (Unknown 포함) | 141,340,000원 | 0 | Unknown으로 표시하지 않으면 16행, 4,015,000원 누락 |
| 월별 | 139,437,000원 | 1,903,000원 | 주문일 누락 주문 2건(상세 6행) |
| 결제수단별 | 141,340,000원 | 0 | - |
 
![핵심 EDA 결과](images/step02_eda.png)
 
### 결과 관찰
completed 매출은 패션이 32,940,000원(290개)으로 가장 많고, 식품이 25,035,000원(226개)으로 뒤를 이었습니다. 표기 차이를 합치면 생활용품 9,109,000원, 뷰티 17,751,000원, 전자기기 21,590,000원이 되어, 뷰티(17,751,000원)가 도서(17,663,000원)보다 높아집니다. 월별 매출은 2026-08이 16,980,000원으로 가장 높았는데, 주문 수는 15건으로 2026-03(13건)과 비슷하지만 주문당 평균은 1,132,000원으로 2026-03(557,462원)의 약 2배였습니다. 결제수단별로는 bank_transfer가 39,461,000원(49건)으로 총매출이 가장 많고, 주문당 평균은 card가 938,550원으로 가장 높았습니다.
 
### 나의 해석과 판단
세 기준으로 같은 completed 금액을 나눴는데 합계가 다르게 나온 것은, 집계 과정에서 빠지는 행이 있다는 뜻입니다. 카테고리는 products에 없는 상품을 참조하는 행이 `groupby`에서 자동으로 빠졌고, 월별은 주문일이 없는 행이 빠졌습니다. 그래서 카테고리 누락 행은 `Unknown`으로 표시해 금액이 사라지지 않게 했고, 주문일 누락은 별도 행으로 기록했습니다.
카테고리 표기 차이는 표준화하지 않았기 때문에, 카테고리 순위와 상품 수·매출 상관계수(0.918, Unknown 제외 10개 카테고리) 모두 표기 차이의 영향을 받습니다. 특히 뷰티와 도서의 순위는 표기를 합치느냐에 따라 바뀌므로, 현재 순위를 확정적인 결과로 보지 않았습니다.
 
### 업무·분석적 의미
합계 검증 없이 카테고리별 표만 보고하면 completed 매출이 4,015,000원 적게 보고됩니다. 월별 표만 보고하면 1,903,000원이 빠집니다. 따라서 보고서에는 어떤 기준으로 집계했고 어떤 행이 빠졌는지를 함께 적어야 합니다. 2026-08처럼 매출이 높은 달은 주문 수보다 주문당 금액이 큰 영향을 준 것으로 보여, 큰 주문이 있었는지 확인하는 것이 다음 단계입니다.
 
### 한계와 추가 확인 사항
- `Unknown` 4,015,000원은 Chapter 05 전처리에서 가격 오류로 제외된 상품(18, 48, 73, 95)과 원본에 없는 상품(9991)을 참조하는 주문입니다. 실제 카테고리를 알 수 없어 카테고리 분석에서 해석할 수 없습니다.
- 미래 날짜 주문(2027-01-15)은 입력 오류일 가능성이 있지만 현재 데이터만으로 확인할 수 없습니다.
- 고객 정보가 없는 completed 주문 상세 1행(376,000원)은 도시별 분석에서 `Unknown(고객 정보 없음)`으로 따로 표시했습니다.
- 할인, 원가·마진, 재고, 프로모션 정보가 없어 매출 차이의 원인이나 인과관계를 판단할 수 없습니다.
## 3. 관찰 → 가설 → 추가 검증
### 사례 1
- 관찰: completed 원본 합계는 141,340,000원이지만, 월별 합계는 139,437,000원으로 1,903,000원 적습니다. 카테고리별 합계도 `Unknown`으로 표시하기 전에는 137,325,000원으로 4,015,000원 적었습니다.
- 가설: 집계 기준 컬럼(카테고리, 주문월)이 비어 있는 행이 `groupby`에서 자동으로 제외되었을 수 있습니다.
- 추가 검증: 차이 금액과 누락 행의 `line_total` 합계를 비교했습니다. 카테고리 누락 16행 4,015,000원, 주문일 누락 6행 1,903,000원으로 차이와 정확히 일치했습니다. 다음으로 누락 행이 특정 상품·기간에 몰려 있는지 확인합니다.
### 사례 2
- 관찰: 패션은 매출 32,940,000원, 판매 수량 290개로 모두 1위입니다. 평균 판매 단가는 113,586원으로 전자기기(122,074원)보다 낮습니다.
- 가설: 패션의 매출 1위는 단가보다 판매 수량의 영향이 클 수 있습니다. 다만 수량이 큰 주문 몇 건의 영향일 수도 있습니다.
- 추가 검증: 카테고리별 주문 상세의 수량 분포(`describe()`, IQR)를 비교하고, 수량이 큰 행을 제외했을 때도 순위가 유지되는지 확인합니다.
### 사례 3
- 관찰: 2026-08 매출은 16,980,000원(15건), 2026-03은 7,247,000원(13건)입니다. 주문 수 차이는 2건이지만 주문당 평균은 1,132,000원과 557,462원으로 약 2배 차이가 납니다.
- 가설: 2026-08의 높은 매출은 주문 수보다 주문당 금액이 큰 주문 때문일 수 있습니다. 계절성이나 행사 효과라고 판단할 정보는 없습니다.
- 추가 검증: 2026-08 주문별 금액 분포와 상위 주문의 카테고리·수량을 확인하고, 주문당 금액 상위 몇 건을 제외했을 때 월별 순위가 어떻게 바뀌는지 비교합니다.
## 4. LLM 질문 확장
- Safe Context: 개인 식별 정보 없이 집계값과 데이터 특성만 사용했습니다.
```text
데이터: Chapter 05 전용 Raw를 scripts/preprocess_data.py로 전처리한 data/processed (개인정보 제외, 집계값만 사용)
주문일 범위: 2025-07-01 ~ 2026-08-30 (단, 2027-01-15, 2028-12-31 주문이 각 1건 있음)
전체 주문: 300건 / completed 주문: 180건 / completed 주문 상세: 446행
completed 매출 합계(line_total): 141,340,000원
 
카테고리별 completed 매출과 판매 수량 (표기 표준화하지 않음, products에 없는 상품은 Unknown):
- 패션: 32,940,000원, 290개
- 식품: 25,035,000원, 226개
- 전자기기: 21,485,000원, 176개
- 도서: 17,663,000원, 174개
- 뷰티: 17,103,000원, 168개
- 스포츠: 13,237,000원, 165개
- 생활용품: 7,001,000원, 108개
- Unknown: 4,015,000원, 34개
- 생활 용품: 2,108,000원, 17개
- 뷰 티: 648,000원, 12개
- 전자 기기: 105,000원, 5개
 
월별 completed 매출:
2025-07 9,561,000 / 2025-08 9,362,000 / 2025-09 5,299,000 / 2025-10 12,629,000
2025-11 9,583,000 / 2025-12 8,581,000 / 2026-01 8,135,000 / 2026-02 7,689,000
2026-03 7,247,000 / 2026-04 11,459,000 / 2026-05 12,006,000 / 2026-06 11,031,000
2026-07 8,966,000 / 2026-08 16,980,000 / 2027-01 909,000
월별 합계는 139,437,000원으로, 주문일이 없는 completed 주문 2건(상세 6행, 1,903,000원)이 빠져 있다.
 
결제수단별 completed 주문 수와 매출:
bank_transfer 49건 39,461,000원 / card 40건 37,542,000원 / kakao_pay 50건 36,861,000원 / naver_pay 40건 27,476,000원
 
데이터 특성:
- category에 '생활 용품', '뷰 티', '전자 기기'처럼 띄어쓰기가 다른 표기가 남아 있다.
- 2027-01은 미래 날짜 주문 1건으로만 구성된다.
- 사용 가능한 컬럼: 주문 상태, 주문일, 결제수단, 상품 카테고리, 수량, 단가, line_total
```
 
- Prompt:
```text
위 Safe Context만 바탕으로 다음 분석에 도움이 될 EDA 질문 3개를 제안해 주세요.
각 질문마다 분석 질문, 필요한 지표, 사용할 데이터 컬럼, pandas로 확인할 수 있는 분석 방법, 결과를 검증하는 방법을 작성해 주세요.
데이터에서 직접 확인되는 사실과 가능한 가설을 구분해 주세요.
현재 데이터에 없는 정보는 추측하지 말고, 계절성·프로모션 효과나 인과관계라고 단정하지 마세요.
```
 
- LLM 제안 요약:
  1. 카테고리 표기를 합치면 매출 순위가 달라지는지 확인 (뷰티 17,751,000원 > 도서 17,663,000원으로 순위 변화)
  2. 카테고리 매출 차이를 판매 수량과 평균 판매 단가로 나눠 보기
  3. 월별 매출 차이를 주문 수와 주문당 평균 금액으로 나눠 보기
- 실제 사용 여부: 수정 후 사용
- 판단 이유: 세 질문 모두 현재 컬럼으로 계산할 수 있고 합계 검증 방법도 포함되어 있습니다. 다만 2번의 "수량 이상값" 기준(IQR)과 3번의 "큰 주문" 기준은 LLM이 정한 것이 아니므로 직접 정해야 하고, 계절성·행사 효과는 현재 데이터로 판단할 수 없습니다.
- 사람이 수정한 내용: 처음 작성한 Safe Context가 이전 데이터 기준(148,990,000원, 184건)이어서 새 데이터 기준으로 바꿨습니다. 카테고리 표기 차이는 표준화하지 않고 관찰로 남기며, products·customers에 없는 행은 `Unknown`으로 따로 표시한다는 기준을 직접 정해 집계에 반영했습니다. 1번 제안의 순위 변화는 노트북에서 직접 계산해 확인했습니다.
![LLM EDA 질문 확장](images/step04_llm.png)
 
## 5. 다음 분석 우선순위
1. 카테고리 누락(`Unknown`)과 주문일 누락 행이 어떤 상품·주문에서 나왔는지 확인하고, 원본에서 복구할 수 있는지 판단합니다.
2. 카테고리별 수량 분포와 평균 판매 단가를 비교해 패션 매출 1위가 수량 때문인지, 큰 수량 주문 몇 건 때문인지 확인합니다.
3. 2026-08 등 주문당 평균이 높은 달의 주문별 금액 분포를 확인해 월별 차이가 큰 주문 때문인지 비교합니다.
### 가장 먼저 확인하고 싶은 이유
누락된 금액(카테고리 4,015,000원, 주문일 1,903,000원)이 남아 있으면 이후 카테고리·월별 분석의 기준 합계가 계속 달라집니다. 어떤 행이 왜 빠졌는지 먼저 정리해야 다른 분석의 결과를 같은 기준으로 비교할 수 있습니다.
 
### 현재 데이터만으로 단정할 수 없는 것
- 카테고리 표기 차이(`뷰 티` 등)가 입력 오류인지, 별도 카테고리인지 단정할 수 없습니다.
- 2026-08의 높은 매출이 계절성이나 행사 효과라고 말할 수 없습니다.
- 결제수단이 주문 금액을 높였다고 말할 수 없습니다. 결제수단별 평균 차이는 주문 구성 차이일 수 있습니다.
- 이익, 재고, 할인 효과는 관련 데이터가 없어 판단할 수 없습니다.
![스크립트 재실행 결과](images/run_eda.png)
 
`python -m scripts.run_eda` 재실행 결과는 노트북 앞부분(`## 12`~`## 14`)과 같은 **전체 주문 상태 기준**이며, `Unknown` 처리가 없어 카테고리가 없는 행은 집계에서 빠집니다. 따라서 completed 기준으로 작성한 이 답안의 금액과는 범위가 다릅니다.
 


![스크립트 재실행- 노트북의 핵심 집계와 같은 범위](images/run_eda.png)

## 최종 체크
- [o] 질문과 지표가 연결되어 있습니다.
- [o] 집계 총합을 검증했습니다.
- [o] 관찰과 가설을 구분했습니다.
- [o] 다음 검증 질문을 작성했습니다.
- [o] LLM 제안을 검증했습니다.
- [o] 최종 Notebook URL 제출 (개인 GitHub 정보 미제공)
