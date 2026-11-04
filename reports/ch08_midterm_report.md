# Chapter 8 중간 프로젝트 보고서

## 1. 분석 목적

온라인 쇼핑몰 데이터를 사용해 완료 주문 기준 매출 현황과 고객 구매 패턴을 분석했습니다.

## 2. 분석 기준

- 매출은 `order_status == "completed"`인 주문만 포함했습니다.
- 취소·환불 주문을 포함한 금액은 전체 주문 상세 금액으로 구분했습니다.
- 고객 이름은 결과와 보고서에서 제외하고 익명화 라벨을 사용했습니다.
- 병합은 관계 검증과 행 수·미매칭 확인을 수행했습니다.

## 3. 데이터 개요

```text
    dataset  rows  columns  missing_values  duplicated_rows
  customers   150        6               0                0
   products   100        4               0                0
     orders   300        5               0                0
order_items   764        5               0                0
```

## 4. 전처리 전후 비교

```text
    dataset  rows_raw  columns_raw  rows_processed  columns_processed
  customers       150            6             150                  6
order_items       764            5             764                  6
     orders       300            5             300                  7
   products       100            4             100                  4
```

## 5. 키와 관계 점검

### 키 중복

```text
    dataset           key  duplicate_count
  customers   customer_id                0
   products    product_id                0
     orders      order_id                0
order_items order_item_id                0
```

### 외래키 관계

```text
                                               check  invalid_count
  orders.customer_id exists in customers.customer_id              0
      order_items.order_id exists in orders.order_id              0
order_items.product_id exists in products.product_id              0
```

### 병합 검증

```text
                  merge  before_rows  after_rows  row_count_preserved  unmatched_count
      order_id → orders          764         764                 True                0
  product_id → products          474         474                 True                0
customer_id → customers          100         100                 True                0
```

## 6. 전체 주문 금액과 완료 주문 매출 구분

```text
                         scope      amount  detail_rows
               all_order_items 255610000.0          764
         completed_order_items 148990000.0          474
excluded_cancelled_or_refunded 106620000.0          290
```

## 7. 카테고리별 완료 주문 매출

```text
category  total_quantity  total_sales  sales_ratio
     스포츠             295     31743000        21.31
    전자기기             259     26400000        17.72
    생활용품             272     23915000        16.05
      뷰티             223     23383000        15.69
      식품             133     16573000        11.12
      도서             149     16389000        11.00
      패션             111     10587000         7.11
```

## 8. 월별 완료 주문 매출

```text
order_month  total_sales  order_count  avg_order_value
    2025-09      2650000            5         530000.0
    2025-10     16970000           19         893158.0
    2025-11     11021000           13         847769.0
    2025-12     21085000           23         916739.0
    2026-01     14272000           16         892000.0
    2026-02      4784000            7         683429.0
    2026-03     18323000           25         732920.0
    2026-04     11546000           16         721625.0
    2026-05     16631000           19         875316.0
    2026-06     12504000           17         735529.0
    2026-07     10788000           15         719200.0
    2026-08      4365000            4        1091250.0
    2026-09      4051000            5         810200.0
```

## 9. 완료 주문 구매 금액 상위 고객

```text
customer_label city  order_count  total_sales  avg_order_value
  Customer 117   성남            5      4100000         820000.0
  Customer 102   고양            4      3996000         999000.0
   Customer 83   수원            4      3880000         970000.0
   Customer 30   서울            5      3590000         718000.0
   Customer 40   서울            4      3523000         880750.0
   Customer 20   인천            2      3191000        1595500.0
    Customer 3   성남            2      3178000        1589000.0
  Customer 111   광주            3      3153000        1051000.0
   Customer 66   서울            4      3093000         773250.0
  Customer 147   부산            2      2990000        1495000.0
```

## 10. 주문 상태별 주문 수

```text
order_status  order_count  order_ratio
   completed          184        61.33
   cancelled           64        21.33
    refunded           52        17.33
```

## 11. 해석 메모

```text
       analysis                          observation                           caution              next_question
 카테고리별 완료 주문 매출    완료 주문 매출 비중이 높은 카테고리를 확인할 수 있습니다. 매출이 높은 이유가 판매 수량인지 단가인지 구분해야 합니다.   카테고리별 평균 판매 단가는 어떻게 다른가?
    월별 완료 주문 매출 시간에 따른 완료 주문 매출의 증가와 감소를 확인할 수 있습니다.     프로모션이나 계절성이 원인이라고 단정할 수 없습니다. 주문 수와 평균 주문 금액 중 무엇이 변했는가?
고객별 완료 주문 구매 금액     완료 주문 구매 금액이 높은 고객군을 확인할 수 있습니다.       일회성 고액 구매와 반복 구매를 구분해야 합니다.    최근 구매일과 구매 빈도는 어떻게 다른가?
    주문 상태별 주문 수       완료, 취소, 환불 주문의 분포를 확인할 수 있습니다.       주문 상태의 정의와 처리 기준을 확인해야 합니다.       취소율과 환불률은 월별로 달라지는가?
```

## 12. 한계점

- 현재 데이터만으로 고객 만족도나 이탈 이유를 분석할 수 없습니다.
- 매출 변동의 원인을 설명하려면 프로모션, 광고, 재고, 계절성 데이터가 필요합니다.
- 완료 주문만 매출로 정의했으며 실제 회계 매출은 결제·배송·반품 정책에 따라 다를 수 있습니다.
- 고객별 결과는 익명화된 분석용 요약이며 개인을 평가하는 용도로 사용하면 안 됩니다.

## 13. 다음 단계

- 카테고리별 판매 수량과 평균 판매 단가를 함께 비교합니다.
- 월별 주문 수와 평균 주문 금액의 변화를 분리해 확인합니다.
- 고객별 최근 구매일과 구매 빈도를 추가합니다.
- 주문 취소율과 환불률의 월별 변화를 분석합니다.
