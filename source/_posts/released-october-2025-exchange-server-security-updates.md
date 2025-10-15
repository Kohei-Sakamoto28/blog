---
title: "2025 年 10 月の Exchange Server のセキュリティ更新プログラムが公開されました"
date: 2025/10/15 15:00:00
lastupdate: 
tags:
- Exchange
---

※ この記事は、[Released: October 2025 Exchange Server Security Updates](https://techcommunity.microsoft.com/blog/exchange/released-october-2025-exchange-server-security-updates/4461276) の抄訳です。最新の情報はリンク先をご確認ください。この記事は Microsoft 365 Copilot および GitHub Copilot を使用して抄訳版の作成が行われています。

Microsoft は、以下の製品に存在する脆弱性に対応するセキュリティ更新プログラム (SU) をリリースしました。

- Exchange Server Subscription Edition (SE)
- Exchange Server 2019
- Exchange Server 2016

以下の Exchange Server のバージョン向けに SU が提供されています：

- Exchange [SE RTM](https://www.microsoft.com/download/details.aspx?id=108422)
- Exchange Server 2019 [CU14](https://www.microsoft.com/download/details.aspx?id=108421) and [CU15](https://www.microsoft.com/download/details.aspx?id=108419)
- Exchange Server 2016 [CU23](https://www.microsoft.com/download/details.aspx?id=108420)

2025 年 10 月のセキュリティ更新プログラム（SU）は、セキュリティ パートナーから責任を持って報告された脆弱性や、Microsoft の内部プロセスによって発見された脆弱性に対応しています。現時点でこれらの脆弱性を悪用した攻撃は確認されていませんが、環境を保護するため、できるだけ早くこれらの更新プログラムを適用することを推奨します。

これらの脆弱性は Exchange Server に影響します。Exchange Online のお客様は、今回のセキュリティ更新プログラムで対応された脆弱性について既に保護されていますので、特別な対応は不要です。ただし、環境内に存在する Exchange サーバーや Exchange 管理ツールをインストールしたワークステーションについては、引き続き更新プログラムの適用を行ってください。

特定の脆弱性 (CVE) に関する詳細は、[Security Update Guide](https://msrc.microsoft.com/update-guide/) (Product Family で "Server Software" を選択してフィルター) をご参照ください。

## Exchange 2016 および 2019 の最後の更新プログラム

2025 年 10 月のセキュリティ更新プログラム (SU) は、Exchange Server 2016 および 2019 向けに一般公開される最後の更新プログラムとなります。今後は、Microsoft アカウント チームへご連絡いただき、[拡張セキュリティ更新プログラム (ESU)](https://techcommunity.microsoft.com/blog/exchange/announcing-exchange-2016--2019-extended-security-update-program/4433495) を取得したお客様のみが、2026 年 4 月まで Exchange 2016 および 2019 向けの新しい更新プログラムを受け取ることができます。

弊社としては、Exchange 2016 および 2019 の ESU を取得するのではなく、[Exchange SE へのアップグレード](https://techcommunity.microsoft.com/blog/exchange/upgrading-your-organization-from-current-versions-to-exchange-server-se/4241305) を行うことを推奨しています。なお、Exchange SE CU2 以降は、Exchange 2016 および 2019 との共存ができなくなることにご注意ください。 (CU2 では、組織内に Exchange 2016 または 2019 が存在する場合、CU2 のインストールがブロックされます)

## Export-ExchangeCertificate による認証証明書 (Auth Certificate) のエクスポートができなくなります

2025 年 10 月のセキュリティ更新プログラム (SU) 以降、セキュリティ強化のため **Export-ExchangeCertificate** コマンドレットを使用した Exchange Server の認証証明書 (Auth Certificate) およびその秘密鍵のエクスポートはブロックされます。 (詳細は [KB5069337](https://support.microsoft.com/help/5069337) を参照してください) 
認証証明書は Exchange Server のワークロードを保護するうえで重要な証明書であり、秘密鍵のエクスポートが必要になることはほとんどありません。認証証明書に関する問題が発生した場合は、[MonitorExchangeAuthCertificate](http://aka.ms/monitorexchangeauthcertificate) PowerShell スクリプトを使用してトラブルシューティングを行ってください。

## 更新プログラムのインストール方法

利用可能な更新パスは以下の通りです。

![](Oct2025SUs.jpg)

- [Exchange Server Health Checker スクリプト](https://aka.ms/ExchangeHealthChecker)を使用して、更新が必要な Exchange サーバーのインベントリを作成し、各サーバーの更新状況 (CU、SU、手動対応) を確認してください。
- 最新の CU をインストールします。[Exchange Update Wizard](https://aka.ms/ExchangeUpdateWizard) を利用して、現在の CU と目標 CU を選択し、手順を確認してください。
- 更新プログラムのインストール後に再度 Health Checker を実行し、追加の対応が必要かどうかを確認します。
- Exchange Server のインストール中やインストール後にエラーが発生した場合は、[SetupAssist スクリプト](https://aka.ms/ExSetupAssist)を実行してください。更新後に問題が発生した場合は、[失敗した Exchange Server の更新プログラムを修正する](https://aka.ms/ExchangeFAQ)や、[2024 年 11 月 SU Exchange Serverインストールしようとするとファイル バージョン エラーが発生する](https://support.microsoft.com/topic/file-version-error-when-you-try-to-install-exchange-server-november-2024-su-a650da30-f8fb-469d-a449-47396cab0a15)もご参照ください。

## よくあるご質問

**弊社の環境は Exchange Online とのハイブリッド構成ですが、対応は必要ですか？**  
Exchange Online は既に保護されていますが、管理目的のみで利用している場合も含め、オンプレミスの Exchange サーバーには今回のセキュリティ更新プログラム (SU) を必ずインストールしてください。SU 適用後に認証証明書を変更する場合は、ハイブリッド構成ウィザードを再実行する必要があります。

**最後にインストールした SU/HU は数か月前のものですが、最新の SU をインストールするためにすべての SU を順番に適用する必要がありますか？**  
すべての SU は累積的です。サポートされている CU を使用している場合、すべての SU や HU を順番にインストールする必要はなく、最新の SU を適用するだけで問題ありません。詳細は[こちらのブログ記事](https://techcommunity.microsoft.com/t5/exchange-team-blog/why-exchange-server-updates-matter/ba-p/2280770)をご参照ください。

**組織内のすべての Exchange Server に SU をインストールする必要がありますか？"Exchange 管理ツールのみ" インストールされたマシンはどうなりますか？**  
<u>すべて</u>の Exchange Server および Exchange 管理ツールがインストールされたすべてのサーバー / ワークステーションに SU を適用することを推奨します。これにより、管理ツールのクライアントとサーバー間の互換性が確保されます。稼働中の Exchange Server が存在しない環境で Exchange 管理ツールのみを更新する場合は、[こちら](https://learn.microsoft.com/exchange/manage-hybrid-exchange-recipients-with-management-tools#update-the-exchange-server-management-tools-only-role-with-no-running-exchange-server-to-a-newer-cumulative-or-security-update)をご参照ください。  

本記事公開時点では、関連するドキュメントが完全には利用できない場合があります。

今後、記事内容に更新があった場合は、こちらに追記します。
