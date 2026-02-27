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
      ・aの初期値:例) 暑がり(+2.0)、標準(0.0)、寒がり(-2.0)  
   2. フィードバック学習  
      ・「暑い」と回答:aにa+0.2 (より低い気温設定にシフト)  
      ・「寒い」と回答：aにa-0.2 (寄り高い気温設定にシフト)
   3. 重ねつけ
      ・湿度が70%以上の時はさらに+1.0℃のオフセットを加える(蒸し暑さ補正)
      ・風速が５m/s以上の時tatはさらに-1.0℃のオフセットを加える(体感の寒さ補正)   
5. 服装提案  
   T(final)に基づいて以下のように提案  
   1. 25℃以上: 夏服(半袖系)  
   2. 20~25℃: 春秋服(長袖系)  
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
   ・Data(Souce): API通信、位置情報、DB通信、Vertex AI(Gemini)によるアドバイス
   lib/  
  presentation/  
    home/  
      home_page.dart (メイン画面)  
      home_vm.dart (状態管理: Riverpod)  
    onboarding/  
      onboarding_page.dart (初期設定)  
      onboarding_vm.dart  
  domain/  
    entities/  
      weather.dart (WeatherData)  
      user_config.dart (UserConfig)  
      clothing.dart (ClothingCategory)  
    usecases/  
      get_clothing_advice_usecase.dart (判定ロジックの心臓部)  
      update_offset_usecase.dart (学習ロジック)  
  data/  
    repositories/  
      weather_repository_impl.dart (Open-Meteo通信)  
      user_repository_impl.dart (Firestore通信)  
    sources/  
      location_service.dart (Geolocator)  
      gemini_service.dart (Vertex AI)  
8. 実装コンポーネント  
   1. データモデル  
      ・WeatherData: 気温、体感温度、湿度、風速、天気コード、時間別リスト  
      ・UserConfig: 感度オフセット、初期体質、花粉症の有無  
      ・ClothingCategory: 推奨される服の種類、レイヤリングの要否  
   2. 外部API  
      ・Open-Meteo: 気象データ取得  
      ・Geolocator: 現在地取得  
   3. データベース・AI設計  
      ・コレクション:users  
      ・フィールド:  
        ・sensitivety_offset:double  
        ・base_preference:String  
        ・last_feedback_at:Timestamp  
      ・Cloud Firestore: ユーザーごとの設定、学習データの保存  
      ・Vertex AI for firebase(Gemini): ロジックが選んだ服に対して、現在の風速や「昨日の感想」を考慮した具体的な着こなしアドバイスを生成  
9. データモデル  
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
       ・UrequiresLayering: 重ね着推奨フラグ(気温差が大きい場合にtrue)  
       ・suggestedItems: 具体的なアイテム例  
10. 運用・拡張プラン  
    ・自分専用のクローゼット連携: ユーザーが自分の持っている服を登録し、「今日はあなたの持っている○○が最適です」と具体的に出してくれる機能  
    ・ヘルスケア連携: 体温や心拍数データから、「今日が少し体調が悪いかもしれないので暖かくしてください」という高度な補正  
    ・SNSへの共有: 「今日の最適スタイル」を画像でシェアできる機能  
11. 開発ステップ  
    1. 基礎機能の実装  
       ・現在地の取得とOpen-Meteoからの気象データの取得  
       ・T(final)計算ロジックの実装と画面への服装カテゴリ表示  
    2. パーソナライズの導入  
       ・Firebase連携による、ユーザーごとのsensitivety_offset保存機能  
       ・フィードバックボタン(熱い/寒い)による学習ループの実装  
    3. AIとUXの強化  
       ・Vertex AI(Gemini)による自然なアドバイス分の生成  
       ・時間軸ロジックに基づいた「帰宅時の冷え込み」アラート機能    
12. UI遷移・画面構成  
    1. 初回起動時  
       ・アプリの趣旨説明  
       ・プロフィール設定: 暑がり、寒がり等の選択(ここでUserConfigのaを決定)  
       ・位置情報・通知の許可  
    2. メイン画面  
       ・現在地と現在時刻  
       ・アドバイスエリア(Geminiによるメッセージ)  
       ・推奨される服装のイラスト、アイコン  
       ・時間軸リスト: 「出発、昼、北区」それぞれの予想転機と服装アイコンの横並び表示  
    3. フィードバック  
       ・フィードバックボタン  
       ・過去の履歴  
       ・完了演出  
    4. 設定画面  
       ・設定変更: 花粉症モードのオン・オフや、基本属性(暑がり等)の再設定
       ・学習データの確認: 現在の自分のaの値を視覚的に表示  
