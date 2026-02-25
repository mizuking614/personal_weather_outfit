# パーソナル服装提案アプリ

1. 目的  
   天気の情報から個人の体感差に合わせた服装を提案してくれる。
2. コンセプト
   一般的な基準ではなく、ユーザーの過去のフィードバックを学修し、使えば使うほど自分専用に進化するパーソナライズ型服装アドバイザー  
3. 提案ロジックの構造
   1. 気象データ:気温、湿度、風速、降水確率、時間ごとの推移。
   2. ユーザー設定・補正値:初期設定とフィードバックによる補正
   3. 時間帯のロジック:出発、滞在、帰宅時などの気温で判定  
4. 動的補正のアルゴリズム
   1. 判定方式
      気象APIから取得した体感温度(T(feellike))に対して、ユーザー固有の感度オフセット(a)を合算して最終判定温度(T(final))を算出  
      T(final) = T(feellike) + a  
      ・aの初期値:例) 暑がり(-2.0)、標準(0.0)、寒がり(+2.0)  
   2. フィードバック学習  
      ・「暑い」と回答:aにa-0.2 (より低い気温設定にシフト)  
      ・「寒い」と回答：aにa+0.2 (寄り高い気温設定にシフト)  
5. 服装提案  
   T(final)に基づいて以下のように提案  
   1. 25℃以上: 夏服(半袖系)  
   2. 20~25℃: 秋冬服(長袖系)  
   3. 16~20℃: 羽織  
   4. 12~16℃: 晩秋（ジャケット等）  
   5. 7~12℃: 冬コート  
   6. 7℃以下: 真冬装備
6. ユーザー体験の差別化
   ・タイムコンテキスト重視:　「最高/最低」ではなく、ユーザーの活動時間帯(例:朝8時と夜18時)のピンポイントな気象データを抽出
   ・重ね着のリコメンド: 1日の寒暖差が10℃を超える場合、脱ぎ記しやすい構成を提示
7. システムアーキテクチャ
   ・presentation(UI): Flutterで画面表示と入力を管理
   ・Domain(Logic): ClothingAdviceUseCaseに判定ロジックを集約
   ・Data(Souce): API通信、位置情報、DB通信
8. 実装コンポーネント
   1. データモデル
      ・WeatherData: 気温、体感温度、湿度、風速、天気コード、時間別リスト
      ・UserConfig: 感度オフセット、初期体質、花粉症の有無
      ・ClothingCategory: 推奨される服の種類、レイヤリングの要否  
   2. 外部API
      ・Open-Meteo: 気象データ取得
      ・Geolocator: 現在地取得
   3. データベース設計
      ・コレクション:users
      ・フィールド:
        ・sensitivety_offset:double
        ・base_preference:String
        ・last_feedback_at:Timestamp  
10. データモデル
    1. WeatherData(気象データ)
       ・currentTemp:　現在の気温
       ・apparentTemp: 現在の体感温度
       ・humidity: 湿度
       ・windSpeed: 風速
       ・weatherCode: 天気状態
       ・hourlyForecast: 24時間分の体感温度リスト(帰宅時の気温判定に使用)
    2. UserConfig(ユーザー設定)
       ・sensitivelyOffset: 感度補正値。初期値は設定に基づいて±2.0、学習により変動  
       ・basePreference: 暑がり、標準、寒がりの初期属性  
       ・hasPollenAllergy: 花粉フラグ  
       ・userId: Firebase Auth と基づく一意のID
    3. ClothingCategory
       ・label: カテゴリー名(「冬コート」、「夏服」など)
       ・requiresLayering: 重ね着推奨フラグ(気温差が大きい場合にtrue)
       ・suggestedItems: 具体的なアイテム例
