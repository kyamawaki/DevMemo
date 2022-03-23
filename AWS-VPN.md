# VPN導通テスト環境構築手順の補足説明

## はじめに

元ドキュメントの内容を山脇が理解した内容で補足説明します。  
ご認識、理解できない点についてご指摘いただけると嬉しいです。

## 構成要素の説明

* VPC(Virtual Private Clound)  
  AWS内に作成された仮想ネットワーク（LANみたいなもの）
* Publicサブネット  
  VPC内のサブネット。インターネットから接続できる。
* Privateサブネット  
  VPC内のサブネット。インターネットから接続できない。
* インスタンス  
  VPC内に配置されるEC2などの実体。
* ACM(AWS Certificate Manager)  
  証明書を管理するサービス
* Elastic IP  
  静的グローバルIPアドレス。  
  AWS内のインスタンスに紐づけることでインターネット経由で紐づいたインスタンスにアクセスできる。  
  検証環境ではNAT Gatwayに紐づけることでプライベートサブネット内のインスタンスとインターネットのデバイスが通信可能となる。  
  - Public IPとの違い  
    Public IPは動的グローバルIPアドレス。  
    インスタンスに紐づけることでインターネットからのアクセスが可能になる。
  ただしインスタンスが起動するごとにIPが割り当てられるので起動ごとに変更される。
* NAT Gateway  
  インスタンスのIPアドレスとNATデバイスのIPアドレスを相互変換するサービス。
* Internet Gateway  
  VPCがインターネットと接続するためのコンポーネント。
* VPN(Virtual Private Network)  
  インターネット上の仮想プライベートネットワーク  
  トンネリング、暗号化、承認などを行うことでセキュリティを高めている  
* Open VPN  
  Open SourceのVPNソフトウェア。  
  プライベート認証局、鍵ペアなどを作成することもできる。
　
## 構築手順

公式チュートリアルを山脇が分かりやすい手順に修正しています。  
各手順の詳細はチュートリアルを参考にしてください。  

1. VPC作成
   1. VPC作成
      1. AWSコンソールを開き、AWSのサービスよりVPCを開く
      2. AWSのサービスよりVPCを選択する
      3. 「VPCを作成」をクリックする
      4. 「IPv4 CIDR」に[10.0.0.0/16]に設定する
      5. 後で識別しやすいようタグをつけておく(eg. VOD)
   2. プライベートサブネットを作成
      1. 左ペインより「サブネット」を選択
      2. 「サブネットを作成」をクリック
      3. さっき作ったVPCに関連付ける
      4. 「サブネット名」を付ける(eg. Private-VOD-Subnet)
      5. [IPv4 CIDR]に[10.0.254.0/24]を設定
   3. パブリックサブネットを作成
      1. 画面下の「新しいサブネット」を選択
      2. 「サブネット名」を付ける(eg. Public-VOD-Subnet)
      3. 「IPv4 CIDR」に[10.0.0.0/24]を設定
1. SSLに使用する証明書と秘密鍵を作成
   1. ツールをインストールする
      1. Open VPNをインストール
      2. EazyRSAを展開
   2. 公開鍵基盤を作成  
   ```./easyrsa init-pki```
   1. ローカル（オレオレ）認証局を作成  
   ```./easyrsa build-ca nopass```
   1. サーバー証明書を作成  
   ```./easyrsa build-server-full server nopass```
   1. クライアント証明書を作成  
   ```./easyrsa build-client-full client1.domain.tld nopass```
   1. 証明書と秘密鍵を適当なフォルダに移動する  
   2. サーバー証明書をACMに登録
1. ローカルPCからVPCにVPN接続
   1. VPN エンドポイントを作成
   2. プライベートサブネットを関連付ける
   3. Route Tableにプライベートサブネットを追加
   4. ローカルPCから接続できるよう認証設定
   5. クライアントVPNエンドポイントの設定ファイルをダウンロードする
   6. ダウンロードしたファイルにクライアントの証明書と秘密鍵を貼り付ける。（DNS名の修正はなくても大丈夫）
   7. AWS VPN Clientアプリをインストール
   8. 修正した設定ファイルを読み込みVPN接続できるか確認する
1. VPC経由でインターネット接続
   1. VPCにInternet Gatewayを作成する
   2. パブリックサブネットにRoute Tableを追加（デフォルトでInternet Gatewayにパケットが飛ぶようにする）
   3. NAT Gatewayをパブリックサブネットに作成する（Elastic IPを割り付ける）
   2. プライベートサブネットにRoute Tableを追加（デフォルトでNAT Gatewayにパケットが飛ぶようにする）
   1. VPN接続した状態でブラウザからCMANに接続する
   表示されるIPアドレスがElastic IPのアドレスなら成功

   