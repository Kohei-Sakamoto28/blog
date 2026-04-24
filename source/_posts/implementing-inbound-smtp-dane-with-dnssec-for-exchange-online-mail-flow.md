---
title: "Exchange Online メール フローの受信 SMTP DANE with DNSSEC の実装"
date: 2026/04/24 10:00:00
lastupdate:
tags:
- Exchange Online
---
※ この記事は、[Implementing Inbound SMTP DANE with DNSSEC for Exchange Online Mail Flow](https://techcommunity.microsoft.com/blog/exchange/implementing-inbound-smtp-dane-with-dnssec-for-exchange-online-mail-flow/3939694) の抄訳です。最新の情報はリンク先をご確認ください。この記事は Microsoft 365 Copilot および GitHub Copilot を使用して抄訳版の作成が行われています。なお、翻訳元の記事の初投稿は 2023 年 9 月 27 日です。

<p style="background: #66FF99">本トピックの最新情報については、<a href=https://techcommunity.microsoft.com/blog/exchange/announcing-general-availability-of-inbound-smtp-dane-with-dnssec-for-exchange-on/4281292>Announcing General Availability of Inbound SMTP DANE with DNSSEC for Exchange Online</a> をご覧ください。</p>

[以前の発表](https://techcommunity.microsoft.com/t5/exchange-team-blog/support-of-dane-and-dnssec-in-office-365-exchange-online/ba-p/1275494)の通り、2024 年 7 月に Exchange Online メール フローの受信 SMTP DANE with DNSSEC のパブリック プレビューをリリースします。これにより、Exchange Online の SMTP DANE with DNSSEC サポートが完成します。送信 SMTP DANE with DNSSEC は [2022 年 3 月](https://techcommunity.microsoft.com/t5/exchange-team-blog/releasing-outbound-smtp-dane-with-dnssec/ba-p/3100920)からサポートされています。

SMTP DANE は、TLS によるメール通信のセキュリティ保護に使用される証明書の正当性を DNS で検証し、TLS ダウングレード攻撃から保護するセキュリティ プロトコルです。DNSSEC は DNS の拡張機能セットで、DNS レコードの暗号的な検証を提供し、DNS スプーフィングや DNS に対する中間者攻撃を防止します。

受信 SMTP DANE with DNSSEC をサポートするため、DNSSEC で保護される Exchange Online 用の新しい DNS インフラストラクチャを構築しました。この新しいアーキテクチャは、Exchange Online の従来の DNS インフラストラクチャ、具体的には現在のお客様の Exchange Online 向けメール フロー用 A レコードをホストしているドメイン mail.protection.outlook.com に影響します。DNS はバックエンド サービスであり「一度設定したら忘れてよい」と考えられがちなため、2024 年 7 月のパブリック プレビューに先立って、メール コミュニティ全体に事前に周知したいと考えています。メール管理者、または Microsoft 365 を再販する、もしくはメール フローの目的で Exchange Online と統合するサードパーティ組織の方は、この記事を確認して、機能リリース前に対応が必要な事項があるかを確認してください。

## SMTP DANE with DNSSEC のリリース方法

受信 SMTP DANE with DNSSEC の初期サポートは、2 つのウェーブで提供されます。

1. **2024 年 7 月: オプトインのパブリック プレビュー。** mail.protection.outlook.com でプロビジョニングされた承認済みドメインに対して、Exchange PowerShell を使用して承認済みドメインで SMTP DANE を有効にし、既存のドメインを新しい DNSSEC 対応ゾーンに移行できるようになります。
2. **2026 年 7 月 1 日: 一般提供 (GA) 後、Exchange Online の受信メール フローに DNSSEC を追加。** 2026 年 7 月に、新しい承認済みドメインのすべての A レコードのプロビジョニングを mx.microsoft 配下の新しいサブドメインに段階的に切り替えます。

この変更は、カスタム ドメインと呼ばれる承認済みドメインのメール フロー用の Mail Exchange (MX) レコードおよび Address (A) レコードのプロビジョニングに関わるもので、特に将来の A レコードをプロビジョニングするゾーンに関するものです。主な変更点は、\<subdomain\>.mx.microsoft というサブドメインの導入で、将来の承認済みドメインの A レコードのホストとして mail.protection.outlook.com を置き換えるものです。DNSSEC のスケール上の制限により、1 つの大規模ドメインではなく多数のサブドメインを使用する必要があります。

**現在の状態:** 現在、承認済みドメインが作成されると、Exchange Online は mail.protection.outlook.com に A レコードをプロビジョニングし、管理者はその A レコードを参照する MX レコードを構成します。これらのレコードについては何も変わりません。たとえば、管理者が 2025 年 4 月より前に Contoso.com を承認済みドメインとして追加した場合、Microsoft 365 管理センターには次のように表示されます。

![](DANE01.jpg)

**変更点:** 2026 年 7 月 1 日以降、新しい A レコードの一部は mx.microsoft 配下の多数のサブドメインのいずれかにプロビジョニングされます。これは 2025 年 4 月から 7 月にかけて段階的に増加し、2025 年 7 月までにすべての承認済みドメインの 100% が mx.microsoft 配下のサブドメインにプロビジョニングされることを目標としています。テナント管理者は、特定のサブドメイン内の正しい A レコードを参照する MX レコードを構成する必要があります。これは、承認済みドメインを初めて手動で構成する際の管理者エクスペリエンスに対する唯一の変更です。2025 年 4 月より前に作成された承認済みドメインについては、受信 SMTP DANE with DNSSEC パブリック プレビューの一環として Exchange PowerShell コマンドレットがリリースされ、DNS レコードを DNSSEC で保護されたドメインに移行するために使用できます。エクスペリエンスを簡素化するために、2025 年中に DNSSEC 有効化ウィザードがリリースされる予定です。

Contoso.com の例で続けると、Contoso.com の管理者が 2025 年 4 月以降に Fabrikam.com を承認済みドメインとして追加した場合 (テナントの onmicrosoft.com ドメインとしてではなく)、Microsoft 365 管理センターには次のように表示される可能性があります。

![](DANE02.jpg)

ドメイン名の '1j2b' の部分はランダムに割り当てられるため、この変更により MX レコードの自動プロビジョニングにあいまいさが生じ、mail.protection.outlook.com の A レコードを参照する MX レコードを自動プロビジョニングする自動化を使用している方に問題が発生します。MX DNS レコードの作成に自動化を使用しているリセラーやお客様は、2025 年 4 月以降、A レコードが常に mail.protection.outlook.com にプロビジョニングされることに依存できなくなります。

これにより、サービスのメール フローに DNSSEC が追加されます。大きなセキュリティ上のメリットがあるため、DNSSEC を無償で提供します。mail.protection.outlook.com は無期限に運用を継続しますが、このドメインへの将来の承認済みドメイン A レコードのプロビジョニングは停止し、SMTP DANE with DNSSEC などの新しい DNS の機能強化は行われません。2024 年 7 月のパブリック プレビューの一環として、Microsoft 365 管理センターや Exchange PowerShell を使用して、メール フローの DNS レコードを mail.protection.outlook.com から mx.microsoft 配下の新しいサブドメインに移行し、特定の承認済みドメインで DNSSEC を有効にしてから、その DNSSEC 対応の承認済みドメインで SMTP DANE を有効にできるようになります。

## 影響

上記の変更に伴い、考慮が必要な事項があります。

- **mail.protection.outlook.com のハードコーディング:** MX ルックアップ、リトライ、検証、フォールバック メカニズムでの mail.protection.outlook.com のハードコーディングは、2024 年 7 月に新しい mx.microsoft サブドメインが導入されて SMTP DANE with DNSSEC パブリック プレビューの一環として採用されると、信頼できなくなります。承認済みドメインの MX レコード構成に関する情報を取得する必要がある場合は、[List serviceConfigurationRecords](https://learn.microsoft.com/graph/api/domain-list-serviceconfigurationrecords?view=graph-rest-1.0&tabs=http) Graph API を使用してください。
- **MX レコードの自動プロビジョニング:** 初期セットアップ時の MX レコードの自動プロビジョニングは、2025 年 4 月までに mail.protection.outlook.com の A レコードの参照を停止する必要があります。MX レコードの自動プロビジョニングは、[List serviceConfigurationRecords](https://learn.microsoft.com/graph/api/domain-list-serviceconfigurationrecords?view=graph-rest-1.0&tabs=http) Graph API を使用して "mailExchange" フィールドを取得する方式に移行する必要があります。これが MX レコードを正しく構成するために必要な正確なドメイン サフィックス情報を提供する唯一の信頼できるソースです。この変更により、対応しなかった場合に将来のテナントのメール フローで発生する可能性のある重大な問題を防止できます。プロビジョニングを変更しない場合、2025 年 4 月以降、A レコードは mx.microsoft ゾーンに作成されますが、MX レコードが誤ったゾーン (mail.protection.outlook.com) を指してしまい、メール フローが機能しなくなります。
- **IP/URL リストの更新:** 標準的な運用手順として、製品ドキュメントと、Microsoft に送信する組織がアクセス コントロール リストで「許可」に含める必要がある IP/URL のリストを更新します。メール サービス プロバイダー、管理者、または Exchange Online に送信するその他のエンティティは、[Office 365 の URL と IP アドレス範囲](https://learn.microsoft.com/microsoft-365/enterprise/urls-and-ip-address-ranges?view=o365-worldwide)のページを追跡し、ネットワークおよびファイアウォールの設定に必要な変更を行ってください。

## 制限事項

受信 SMTP DANE with DNSSEC のサポートに向けて、パブリック プレビューでは初期段階でサポートされないシナリオがいくつか確認されています。

- **Viral またはセルフサービス サインアップ ドメイン:** Microsoft から購入し、セルフサービスとしてセットアップされたドメインは、パブリック プレビューの一環として受信 SMTP DANE with DNSSEC のサポート対象にはなりません。将来的にこれらのドメインでの受信 SMTP DANE with DNSSEC のサポートを予定していますが、ETA は未定で、暫定的に 2024 年下半期を目指しています。
- **onmicrosoft.com ドメイン:** テナントの onmicrosoft.com ドメインは、パブリック プレビューの一環として受信 SMTP DANE with DNSSEC のサポート対象にはなりません。onmicrosoft.com ドメインでの受信 SMTP DANE with DNSSEC のサポートを調査中ですが、ETA は未定です。
- **サードパーティ ゲートウェイ:** テナント宛てのメールを受け取り、処理してから Exchange Online にリレーするサードパーティのメール ゲートウェイを受信パスで使用しているお客様は、異なる有効化プロセスに従う必要があり、構成を成功させるためにサードパーティのサービス プロバイダーとの連携が必要になる場合があります。この機能はこのようなお客様でも使用できますが、保護されるのはサードパーティ ゲートウェイから Exchange Online へのトラフィックのみで、サードパーティが送信 SMTP DANE with DNSSEC の検証をサポートしている場合に限られます (サードパーティが Exchange Online に対して SMTP DANE および DNSSEC の検証を行うため)。サードパーティが送信 SMTP DANE with DNSSEC をサポートしていない場合もフローは動作しますが、サードパーティがプロトコルをサポートするまでセキュリティ上の追加のメリットはありません。Microsoft 365 管理センターの DNSSEC 有効化ウィザードでは、この構成のセットアップはサポートされません。サードパーティのメール ゲートウェイを使用しているお客様は、Exchange PowerShell と提供されるドキュメントを使用して受信 SMTP DANE with DNSSEC をセットアップする必要があります。
- **完全委任ドメイン:** Microsoft がホストし、NS 委任を使用して Microsoft のネーム サーバーがドメインに対して権威を持つドメインは、パブリック プレビューの一環として受信 SMTP DANE with DNSSEC のサポート対象にはなりません。将来的にこれらのドメインでの受信 SMTP DANE with DNSSEC のサポートを予定しており、暫定的に 2025 年下半期を目指しています。
- 今後、新しい制限事項が追加される可能性があります。

## フィードバック

ここで取り上げていない懸念や依存関係がある場合は、フィードバックをお寄せください。フィードバックや懸念がある場合は、[原文の記事](https://techcommunity.microsoft.com/blog/exchange/implementing-inbound-smtp-dane-with-dnssec-for-exchange-online-mail-flow/3939694)にコメントしてください。必要に応じて直接ご連絡します。

変更が難しい場合もありますが、今回の変更は Exchange Online のメール フローの円滑な運用を維持するために必要なものです。Microsoft 365 のメール配信のセキュリティと信頼性を向上させるための取り組みに、ご理解とご協力をお願いします。
