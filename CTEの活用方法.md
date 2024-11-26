### 3. モデルのベストプラクティス

#### CTEの活用方法

Common Table Expression（CTE）は、SQLクエリ内で一時的な名前付き結果セットを定義する機能です。CTEを適切に活用することで、クエリの可読性を向上させ、複雑なロジックを分解して管理しやすくすることができます。本ガイドラインでは、CTEの効果的な使用方法とベストプラクティスを以下に示します。

---

##### 1. **CTEのメリット**
- **可読性の向上**  
  複雑なクエリを段階的に構築することで、コードが簡潔で理解しやすくなります。
  - 例: データのフィルタリング → 集計 → 結果の計算という処理を分けて記述。
- **再利用性の向上**  
  同じサブクエリを複数回使用する必要がある場合、CTEを用いることで冗長なコードを排除できます。
- **デバッグの容易さ**  
  各ステップを個別にテストすることで、エラーを迅速に特定できます。

---

##### 2. **CTEの基本構文**
CTEは、`WITH`句を使用して定義します。複数のCTEをカンマで区切り、`SELECT`文で参照します。

```sql
WITH cte_name AS (
    SELECT
        column1,
        column2
    FROM
        table_name
    WHERE
        condition
)
SELECT
    *
FROM
    cte_name
```

---

##### 3. **CTEを利用した段階的なクエリ構築**
CTEを使用してクエリを段階的に構築します。
- **例: 売上集計クエリ**
  - ステップ1: 注文データのフィルタリング
  - ステップ2: 顧客ごとの売上を集計
  - ステップ3: 結果の並び替え

```sql
WITH filtered_orders AS (
    SELECT
        order_id,
        customer_id,
        order_total
    FROM
        orders
    WHERE
        order_status = 'completed'
),
customer_sales AS (
    SELECT
        customer_id,
        SUM(order_total) AS total_sales
    FROM
        filtered_orders
    GROUP BY
        customer_id
)
SELECT
    customer_id,
    total_sales
FROM
    customer_sales
ORDER BY
    total_sales DESC
```

---

##### 4. **複数のCTEを活用する際のルール**
- **意味のある名前を付ける**  
  CTE名は、その内容や役割を簡潔に表現する名前を付けます。
  - 良い例: `filtered_orders`, `customer_sales`
  - 悪い例: `cte1`, `temp_table`
- **適切な長さで分割**  
  CTEが冗長にならないよう、1つのCTEに1つの責任（フィルタリング、集計、結合など）を持たせます。

---

##### 5. **ネストの回避**
CTEを使うことでサブクエリのネストを避け、クエリの構造を平坦化します。これにより、コードの可読性が向上します。
- 良い例:
  ```sql
  WITH filtered_data AS (
      SELECT ...
  ),
  aggregated_data AS (
      SELECT ...
  )
  SELECT ...
  FROM aggregated_data
  ```
- 悪い例（ネストしたサブクエリ）:
  ```sql
  SELECT ...
  FROM (
      SELECT ...
      FROM (
          SELECT ...
          FROM ...
      ) AS subquery
  ) AS subquery
  ```

---

##### 6. **パフォーマンスへの配慮**
CTEの使用は可読性を高める一方で、パフォーマンスに影響する可能性があります。
- **不要なCTEの作成を避ける**  
  必要以上にCTEを分割すると、データベースの最適化が妨げられる場合があります。
- **素材データのフィルタリングを早期に行う**  
  CTEの初期段階でデータセットをフィルタリングすることで、後続の処理の負荷を軽減します。

---

##### 7. **再利用可能なロジックのマクロ化**
頻繁に使用するCTEロジックは、マクロとして定義し、複数のモデルで再利用します。
- 例: 有効な注文データを抽出するCTEロジック
  ```sql
  {% macro get_active_orders() %}
  WITH active_orders AS (
      SELECT
          order_id,
          customer_id,
          order_total
      FROM
          orders
      WHERE
          order_status = 'active'
  )
  SELECT * FROM active_orders
  {% endmacro %}
  ```

---

このようにCTEを適切に活用することで、SQLコードの可読性を高め、複雑なロジックを効率的に構築することができます。CTEの使用ルールを遵守することで、プロジェクト全体で一貫性を保ちながら、効率的な開発を実現します。