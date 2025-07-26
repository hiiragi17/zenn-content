---
title: "複雑な削除制限をLaravelのPolicyで統一管理する。"
emoji: "🛍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ 'laravel', 'policy']
published: false
---

# 始めに

とある案件でリファクタリングを行なっていたとき、
複雑な削除制限をLaravelのPolicyで統一しているのを見ました。
以前Policyの記事を書いたのですが、
その時は認可処理で使用していました。
てっきりPolicyとはそれだけの機能しかないと思っていたのですが、
使い方次第で色々できそうなことがわかりました。
今回はその使い方をまとめます。

https://zenn.dev/arsaga/articles/b5a28d65075322

# 今回の例

今回は下記のようなとあるECサイトの商品カテゴリ管理のER図があるとします。
![Policy ERダイアグラム](/images/delete_setting_policy/Policy_ER.png)

```php
// Categoryモデル
class Category extends Model
{
    public function products()
    {
        return $this->hasMany(Product::class);
    }
    
    public function campaigns()
    {
        return $this->hasMany(Campaign::class);
    }
    
    public function recommendations()
    {
        return $this->hasMany(Recommendation::class);
    }
    
    public function searchFilters()
    {
        return $this->hasMany(SearchFilter::class);
    }
}
```

商品カテゴリなので、
商品との紐づきが勿論あり。
その後キャンペーンとかでもカテゴリが使用されるようになり、
おすすめ商品設定でも使用されるようになり。
検索フィルターでも使用されるようになりました。
他にも色々と使用される可能性が今後もありますね。
当然ですが商品カテゴリを消す場合、
これらと紐づきがないかを確認してから消すようにしないといけないですね。
そうでないと存在しない商品カテゴリと紐づいてしまうことになって、
本番環境でエラーが出てしまいます。
(そして何故かわからないけれど、意外に見落とされがちな気がしました)

# これまでの実装

これまでだったらシンプルに、
Controllerに全部集約する方法があります。

```php
public function destroy(Category $category)
{
    // 削除制限チェック
    if ($category->products()->exists() ||
        $category->campaigns()->exists() ||
        $category->recommendations()->exists() ||
        $category->searchFilters()->exists()) {
        return back()->with('error', 'データと紐づいているため削除できません');
    }
    
    $category->delete();
    return redirect()->route('categories.index')
        ->with('success', '削除しました');
}
```

そもそも上の、
`$category->products()->exists()`でむっちゃ簡単に存在確認できるな、
Laravelはわかりやすいなあって今更感動したりしています。
LaravelのEloquent ORMって非常に有難いですよね。
リレーションを貼るだけで、

```php
class Category extends Model
{
    public function products()
    {
        return $this->hasMany(Product::class);
    }
}
```

下記のようにメソッドを簡単に呼び出せますからね。

```php
// 動的プロパティアクセス（リレーションの結果を取得）
$category->products  // Collection を返す

// メソッド呼び出し（クエリビルダーを返す）
$category->products()->exists()  // HasMany を返す
```

FastAPIでSQLAlchemyを使用していたときは、
明示的ではありますが冗長なコード書かないと、
同じようなことができないですし。
多分SQLAlchemyならこんな感じ。

```python
has_products = session.query(exists().where(Product.category_id == category_id)).scalar()
```

リレーションを定義するだけでクエリ機能が使用できるの、やっぱ強すぎるな。
と今更Laravelの良さを再確認しているところです。

https://readouble.com/laravel/11.x/ja/eloquent-relationships.html

https://readouble.com/laravel/8.x/ja/collections.html

話がずれましたがコントローラーに全部集約するだけで、
依存関係のあるテーブルが全部確認できて、
1つでも存在するカテゴリがあれば削除ができなくなります。
まあなんかこれでもいい気がするのですが、
ここではPolicyを使用して実装します。

# Policy使用

```php
// Controller
public function destroy(Category $category)
{
    $this->authorize('delete', $category);
    
    $category->delete();
    return redirect()->route('categories.index')
        ->with('success', '削除しました');
}

// Policy
public function delete(User $user, Category $category)
{
    return !($category->products->count() ||
            $category->campaigns->count() ||
            $category->recommendations->count() ||
            $category->searchFilters->count());
}
```

# Policyを使うメリット

コントローラーでの実装でも小規模なら機能しますが、
Policyを使うことで以下のメリットがあると考えられます。

## 1.再現性の向上

```php
// コントローラー
if (Auth::user()->can('delete', $category)) {
    // 削除ボタンを表示
}

// Bladeテンプレート
@can('delete', $category)
    <button class="btn btn-danger">削除</button>
@endcan

// API
if ($user->can('delete', $category)) {
    return ['can_delete' => true];
}
```

同じ削除制限ロジックを複数箇所で簡単に再利用できます！

## 2.責任の分離

```php
// Controller: HTTPリクエスト処理に専念
public function destroy(Category $category)
{
    $this->authorize('delete', $category); // 認可は委譲
    $category->delete();                   // ビジネスロジックに集中
    return redirect()->route('categories.index')->with('success', '削除しました');
}

// Policy: 削除可能性の判定に専念
public function delete(User $user, Category $category)
{
    // 削除制限ロジックのみに集中
    return !($category->products()->exists() || ...);
}
```

コントローラーに責任が混在していたので、
単一責任原則を侵していた構造になっていました。
そのためPolicyに削除制限ロジックを委譲し、
コントローラーにHTTP処理責任だけを任せることで、
設計によりコードを変更する場合の影響範囲が狭くなりますし、
拡張性・再利用性も向上しますね。

## 3.テストもしやすい

```php
// Policyの単体テスト
public function test_cannot_delete_category_with_products()
{
    $category = Category::factory()->create();
    Product::factory()->create(['category_id' => $category->id]);
    
    $policy = new CategoryPolicy();
    $this->assertFalse($policy->delete(User::factory()->create(), $category));
}

// vs コントローラーの統合テスト（より複雑）
public function test_cannot_delete_category_with_products()
{
    $category = Category::factory()->create();
    Product::factory()->create(['category_id' => $category->id]);
    
    $response = $this->delete("/categories/{$category->id}");
    $response->assertRedirect();
    $this->assertDatabaseHas('categories', ['id' => $category->id]);
}
```

Policyの単体テストの方がシンプルです。

# 他のアプローチとの比較

これまで私はLaravelと言えば、
Repository+Serviceパターンだと思っていました。
もしそれで実装するなら下記のようになりますかね。

```php
// CategoryRepository
public function hasProducts($categoryId)
{
    return Product::where('category_id', $categoryId)->exists();
}

// CategoryService  
public function delete($categoryId)
{
    if ($this->categoryRepository->hasProducts($categoryId)) {
        throw new \Exception('商品が紐付けられているため削除できません');
    }
    // ...
}
```

元々はコントローラーに全部ロジックが書いてありました。
それをリファクタリングしようとなって、
どのような設計にするか考えたときに、
ServiceとRepositoryの設計の話も出たのですが。
今回の設計だと過剰設計になって、
あまり良くない設計なのでなしになりました。

# まとめ

Policyは認可処理だけでなく、
削除制限のようなビジネスルールの管理も活用できるのだと学びました。
でもこれPolicyが強いっていうより、
Eloquent ORMが強いって話ですね。
まあどちらにしろ、適切なアーキテクチャを選択して、
保守性が高くて拡張しやすいコードを書けるようになっていきたいですね。
それではここまで読んでいただきありがとうございました。
誰かの参考になれば幸いです。

# 参考文献

この記事は以下の記事を参考にして執筆させていただきました。

https://readouble.com/laravel/11.x/ja/eloquent-relationships.html

https://readouble.com/laravel/8.x/ja/collections.html
