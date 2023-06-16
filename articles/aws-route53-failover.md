【aws】route 53を使った、フェイルオーバー構成の実装と運用方法
aws,route53,フェイルオーバー
aws-route53-failover

## aws route 53とは何か？フェイルオーバーの概要について

こんにちは。今回は、awsについて初心者エンジニアに向けて、route 53を使ったフェイルオーバー構成の実装と運用方法についてお伝えします。

まずは、aws route 53とは何かをご説明します。route 53は、amazon web servicesが提供するdnsサービスです。dnsとは、ドメイン名（www.example.com）をipアドレス（192.0.2.1）に変換する仕組みのことで、route 53はこのdnsサービスをクラウド上で提供しています。

route 53は、ユーザーがドメイン名に紐付けたリソース（ec2インスタンスやs3バケットなど）を、そのリソースが稼働しているリージョンやaz（アベイラビリティーゾーン）に応じて、自動で最適な場所にルーティングする機能を持っています。また、route 53は、ヘルスチェック機能を備えており、リソースの可用性を自動でチェックし、異常が検出された場合には自動でルーティングすることができます。

今回は、このroute 53を使ってフェイルオーバー構成を実現する方法についてお伝えします。

## フェイルオーバー構成の設計と実装方法について

フェイルオーバーとは、1つのサービスに対して複数のサーバーを用意し、あるサーバーがダウンした場合には他のサーバーに自動で切り替わる仕組みのことです。フェイルオーバーを実現することで、サービスの可用性を高めることができます。

では、route 53を使ってフェイルオーバー構成を実現するためには、どのような設計が必要でしょうか。まずは、以下の2つのレコードを用意します。

- プライマリーレコード：通常のリソースへの問い合わせを受け付けるためのレコード
- セカンダリーレコード：フェイルオーバーが発生した場合にアクティブになるレコード

プライマリーレコードには、通常通りの設定を行います。セカンダリーレコードには、プライマリーレコードと同様の設定を行いますが、routing policyを「failover」と設定し、health checkを適用します。

さらに、フェイルオーバーが発生した場合には、セカンダリーレコードがアクティブになり、アクティブになった状態で問い合わせが行われるようにする必要があります。そのためには、以下の手順が必要です。

1. プライマリーレコードとセカンダリーレコードを作成する。
2. プライマリーレコードを正常な状態にしておく。
3. セカンダリーレコードのhealth checkに、プライマリーレコードのアドレスを指定する。
4. セカンダリーレコードをfailoverする。

failoverした場合は、適切なロードバランサーやec2インスタンスなどのリソースに振り分けられるよう、必要に応じて設定を行ってください。

## route 53 health checkを使ったサーバーのヘルスチェックとフェイルオーバーの自動化

次に、route 53 health checkを使ったサーバーのヘルスチェックとフェイルオーバーの自動化についてお伝えします。

route 53 health checkは、awsが提供するヘルスチェック機能で、ec2インスタンスやelbなどのリソースに対して、http、tcp、https、dnsなどのプロトコルに基づいたヘルスチェックを行うことができます。また、このヘルスチェック結果に応じて、自動でルーティングすることができます。

フェイルオーバーの自動化を行うには、まずroute 53 health checkでリソースのヘルスチェックを設定します。例えば、ec2インスタンスへのヘルスチェックを設定する場合は、以下の手順で行うことができます。

1. awsコンソールから、route 53 health checkを開く。
2. 「create health check」をクリックし、必要事項を入力する。
3. 「create health check」をクリックして、ヘルスチェックを作成する。

ヘルスチェックが作成されたら、route 53の管理画面からレコードセットを変更し、そのレコードセットに対してヘルスチェックを指定します。

指定したヘルスチェックが失敗した場合には、自動でフェイルオーバーを行うようにroute 53を設定することができます。例えば、route policyを「failover」に設定し、primaryとsecondaryのレコードをそれぞれ作成しておきます。この状態で、primaryのヘルスチェックが失敗した場合には、自動でsecondaryにフェイルオーバーするように設定することができます。

## route 53のエイリアスレコードを使ったフェイルオーバーの設定方法

次に、route 53のエイリアスレコードを使ったフェイルオーバーの設定方法についてお伝えします。

エイリアスレコードは、awsが提供するdnsサービスの特徴で、他のdnsサービスとは異なり、awsが管理するリソースに対して直接的にルーティングを行うことができます。このエイリアスレコードを使うことで、常に最適なリージョンやazに対してルーティングを行うことが可能です。

エイリアスレコードを使ったフェイルオーバーの場合は、「raf（routing to another failover）」方式を使用します。この方式は、セカンダリーリソースを別のエイリアスレコードで指定し、フェイルオーバーが発生した場合には、自動的にセカンダリーリソースを使うようにルーティングする方式です。

具体的な手順は以下の通りです。

1. プライマリーリソースのエイリアスレコードを作成する。
2. セカンダリーリソースのエイリアスレコードを作成する。
3. エイリアスレコードを使ったraf方式を選択して、フェイルオーバーを実施する。

この方法では、フェイルオーバーが発生した場合には自動的にセカンダリーリソースにルーティングされます。また、raf方式を使うことで、フェイルオーバーが発生しない場合でも、常に最適なリージョンやazに対してルーティングを行うことができます。

## フェイルオーバー構成のテスト方法と、ユーザーに与える影響について

次に、フェイルオーバー構成のテスト方法と、ユーザーに与える影響についてお伝えします。

フェイルオーバー構成を実施する場合には、常にユーザーに最小限の影響を与えるように注意が必要です。特に、フェイルオーバーが発生した場合には、リソースの切り替わりに伴ってユーザーに不具合が生じる可能性があるため、十分にリスクを把握した上でテストを実施する必要があります。

フェイルオーバー構成のテストには2つの方法があります。

1. ヘルスチェック機能を一時的に無効にして、手動でセカンダリーリソースに切り替える。
2. ヘルスチェック機能を使って、自動でセカンダリーリソースに切り替える。

手動で切り替える場合には、事前に各リソース（ec2インスタンスやlbなど）を用意しておく必要があります。また、手動で切り替えた場合には、その後ヘルスチェックを再度有効にして、正しくフェイルオーバーが発生するかどうか確認する必要があります。

自動切り替えを使う場合には、セットアップに時間がかかる場合があるため、事前にテストを実施しておくことが重要です。また、自動切り替えされた場合に、問題なくシステムが動作するかどうかを確認する必要があります。

## route 53のフェイルオーバー構成の監視と運用方法のコツ

最後に、route 53のフェイルオーバー構成の監視と運用方法のコツについてお伝えします。

フェイルオーバー構成を実施する場合には、定期的な監視が必要です。特に、route 53のヘルスチェック機能を使って自動的に切り替えが実施される場合には、障害発生から切り替え完了までに時間がかかる場合があるため、この間に問題が発生することがあります。

そのために、フェイルオーバーが発生した場合には、自動的に通知を行い、誰かが手動で対応できるようにしておくことが重要です。また、フェイルオーバー構成を運用する場合には、運用マニュアルや監視ポイント、運用の責任者などを定めておくことが必要です。

以上が、route 53を使ったフェイルオーバー構成の実装と運用方法についての説明でした。aws route 53は、フェイルオーバー構成の実装に必要な機能を備えたdnsサービスであり、高い可用性を確保するためには必要不可欠です。是非、活用してみてください。

## 参考文献

- [aws 公式ドキュメント - amazon route 53 ドキュメント](https://aws.amazon.com/jp/route53/)
- [amazon route 53を使ったフェイルオーバーネットワークの作成手順](https://aws.amazon.com/jp/getting-started/hands-on/configure-failover-using-amazon-route-53/)
- [amazon route 53 ヘルスチェック機能](https://aws.amazon.com/jp/route53/health-checks/)


## AWS Route53関連のまとめ
https://hack-note.com/summary/aws-route53-summary/


## オンラインスクールを講師として活用する！
https://hack-note.com/programming-schools/


## 0円でプログラミングを学ぶという選択
- [techacademyの無料体験](//af.moshimo.com/af/c/click?a_id=2612475&amp;p_id=1555&amp;pc_id=2816&amp;pl_id=22706&amp;url=https%3a%2f%2ftechacademy.jp%2fhtmlcss-trial%3futm_source%3dmoshimo%26utm_medium%3daffiliate%26utm_campaign%3dtextad)
- [オンラインスクール dmm webcamp pro](//af.moshimo.com/af/c/click?a_id=2612482&amp;p_id=1363&amp;pc_id=2297&amp;pl_id=39999&amp;guid=on)
