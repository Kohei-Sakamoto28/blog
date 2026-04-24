---
title: "Exchange Online メール フローの DNS セキュリティの最新化: DNSSEC、SMTP DANE、MTA-STS の最新情報"
date: 2026-04-24 11:00:00
lastupdate:
tags: Exchange Online
---
※ この記事は、[Modernizing DNS Security for Exchange Online Mail Flow](https://techcommunity.microsoft.com/blog/exchange/modernizing-dns-security-for-exchange-online-mail-flow/4514248) の抄訳です。最新の情報はリンク先をご確認ください。この記事は Microsoft 365 Copilot および GitHub Copilot を使用して抄訳版の作成が行われています。

DNS (Domain Name System) プロトコルは、クライアントがインターネット上でメール サーバーを見つけるために使用されます。DNS は既定では暗号化も認証もされていないため、スプーフィング、改ざん、中間者攻撃に対して脆弱です。脅威アクターがメール配信の基盤レイヤーをますます標的にするようになる中、最新の DNS セキュリティ プロトコルは組織を保護するために不可欠なものとなっています。

これらのギャップに対処するため、Exchange Online は DNSSEC、SMTP DANE、MTA-STS といった最新の標準ベースの DNS セキュリティに大きく投資し、可能な限り既定で検証済みの暗号化された改ざん耐性のあるチャネルでメールが配信されるようにしています。

この記事では、これらの取り組みに関する最新情報を提供し、メール セキュリティの水準をさらに高めるための今後の計画について説明します。

### Exchange Online 向け DNSSEC 有効化ウィザード

SMTP DANE with DNSSEC の導入を簡素化するために、2026 年第 3 四半期に Exchange 管理センターで **DNSSEC 有効化ウィザード**をリリースします。このガイド付きワークフローでは、以下を行います。

- DNS の前提条件の検証
- テナント固有の DNSSEC 対応メール フロー エンドポイントのプロビジョニング
- MX 移行時の構成リスクの低減
- SMTP DANE 導入に向けたドメインの準備

SMTP DANE with DNSSEC を完全に適用したい場合は、DNSSEC の有効化が完了した後、[Set up inbound SMTP DANE with DNSSEC](https://learn.microsoft.com/purview/how-smtp-dane-works#set-up-inbound-smtp-dane-with-dnssec) に従って PowerShell で SMTP DANE を有効にします。

### 送信コネクタでの SMTP DANE および MTA-STS 検証の制御

[2026 年 2 月下旬にロールアウトが開始](/blog/announcing-smtp-dane--mta-sts-connector-modes-in-exchange-online/)された新しい機能により、管理者は送信コネクタを介して送信されるメッセージに対する SMTP DANE および MTA-STS の検証動作を明示的に制御できるようになりました。

New/Set/Get-OutboundConnector の MtaStsMode パラメーターと SmtpDaneMode パラメーターを使用すると、コネクタごとに Exchange Online がこれらのセキュリティ プロトコルをどの程度厳格に適用するかを選択できます。

- **Opportunistic (既定):** Exchange Online は SMTP DANE および/または MTA-STS の検証を試行しますが、宛先がサポートしていない場合でもメールを配信します。
- **None:** MTA-STS と SMTP DANE の両方に適用され、検証を完全に無効にします。これにより、ダウングレード攻撃やスプーフィングされた MX リダイレクトを防止するために設計された MTA-STS および/または SMTP DANE の保護が削除されるため、そのコネクタを介して送信されるメールのセキュリティが低下します。
- **Mandatory (SMTP DANE のみ):** SMTP DANE with DNSSEC の完全な検証を強制し、検証に失敗した場合または宛先ドメインが SMTP DANE with DNSSEC をサポートしていない場合はキューイング期間の終了までメールをキューに格納 (その後拒否) します。

この送信コネクタ機能により、パートナー エコシステムとの互換性を維持しながら、より強力な DNS ベースの保護を段階的に導入しやすくなります。

#### DNSSEC 対応メール フロー レコード (A/AAA) の自動プロビジョニングはどうなりましたか?

内部インフラストラクチャ プロジェクトの影響で、この DNS プロビジョニングの変更は 2026 年下半期まで延期する必要がありました。新しい承認済みドメインに対するすべての A レコードのプロビジョニングを mx.microsoft 配下の新しいサブドメインへ徐々に切り替えることは引き続き優先事項ですが、インフラストラクチャの変更は複雑です。大きな課題があったため、サービスの正常性と信頼性を維持しながらこの変更を完了するために必要な作業の順序を見直す必要がありました。

元の発表: [Exchange Online メール フローの受信 SMTP DANE with DNSSEC の実装](/blog/implementing-inbound-smtp-dane-with-dnssec-for-exchange-online-mail-flow/)

### mail.protection.outlook.com に対する更新予定はありますか?

現在、メール フロー ドメイン mail.protection.outlook.com で DNSSEC を有効にする予定はありません。受信メールに DNSSEC が必要な場合は、引き続き mx.microsoft 内の DNSSEC 対応の専用サブドメインに移行する必要があります。MX の変更は運用上デリケートな作業となる可能性があるため、この変更の負担を軽減するために DNSSEC 有効化ウィザードを構築しました。

2026 年第 3 四半期の早い段階で、mail.protection.outlook.com は TCP および EDNS のサポートを受ける予定です。この最新化により信頼性が向上し、クラウド スケールでの将来のセキュリティ強化が可能になります。

### セキュリティの水準を共に高める

これらの投資全体を通じた目標はシンプルです。運用上の複雑さやオーバーヘッドを増やすことなく、強力なメール セキュリティを既定にすることです。DNSSEC、SMTP DANE、MTA-STS は、グローバルなメール エコシステムにおける長年の弱点に直接対処するものであり、Exchange Online はこれらの基盤的な保護を大規模に展開する業界のリーダーであり続けることにコミットしています。

DNS インフラストラクチャの最新化、ドメイン移行のためのより安全なツールの提供、プロトコル適用に対するよりきめ細かい制御の提供を通じて、すべての Exchange Online 利用者のセキュリティの水準を引き続き高めていきます。最新の DNS セキュリティの導入がこれまで以上に容易になります。

Microsoft 365 Messaging Team
