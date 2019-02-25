# DApps開発

# 概要

ropstenにデプロイしたInaCoin（ERC20トークン）を用いたDAppsの開発を行った。インターフェースとしてSlackを利用している。

以下のリポジトリを参考に必要な部分を修正して実装しています。基本機能はこちらを参照ください。

https://github.com/keyiiiii/internal-coin-slack-bot



# 環境

今回はdockerで以下の環境を作って実行しました。Windowsで最初試みたもののweb3がどうしても上手くインストール出来ず。古いバージョンだと入るけど、文法が変わっているので、参考にしているコードをそのまま使えず断念。

- Linux(Ubuntu)
- Node.js(v11.8.0)
- npm(v6.5.0)
- botkit(v0.6.14)
- truffle-hdwallet-provider(v0.0.5)
- web3(v1.0.0-beta.34)



# Dockerでの環境構築について

とりあえず使い方がよく分からなくても、以下のコマンドを打っていけば、環境作って実行することはできるようになります。

コマンド打つ前にdockerのインストールだけは済ませておいてください。



```
$ docker run -itd --name test(ここは好きな名前) -v C:/Users/XXXXXX(自分のユーザー名)/dev/dockermount(ここも好きなフォルダでOK):/tmp node
```

これで`node`の`image`ファイルの`pull`からコンテナの立ち上げまでできます。またローカルにおいているフォルダをマウントしているので、コンテナのbashからローカルフォルダをいじれるようになります。



```
$ docker exec -it test /bin/bash
```

これでコンテナ上のbashが立ち上がります。あとは普通にターミナルを操作していけばOK。



今回のDApps開発ではいくつかライブラリを使用しているので、プロジェクトフォルダまで移動してから、以下のコマンドでインストールを行う。

```
$ npm init -f
$ npm botkit@0.6.14
$ npm truffle-hdwallet-provider@0.0.5
$ npm web3@v1.0.0-beta.34
```

とりあえず動かして上手くいくことを確認したかったので、バージョンを指定してインストールしています。



これで環境の準備は整ったので、あとは以下のコマンドでSlackとropstenをつなぐハブとなるnodejsが立ち上がります。

```
$ node index.js
```

また`index.js`ファイルについては後述。



# Slackbot

DAppsのインターフェイスとしてSlackのbotを利用します。SlackではカスタムインテグレーションでBotsを設定できます。あとでSlackbotへ接続するためのアクセスキーを使用するので、設定してキーを控えておきます。



参考にしたコードはbotkitをカスタマイズしたものになります。単純なbotとして利用する場合は、`index.js`ファイルだけ書き換えればそれっぽいものができます。



今回はweb3を使って、ropsten上のコントラクトを操作する必要があるので、`contract.js`と`abi.js`を追加して作成しています。もちろん全部`index.js`に書いてもよい。



`contract.js`

```javascript
const HDWalletProvider = require("truffle-hdwallet-provider");
const Web3 = require('web3');
const config = require('./config');
const ABI = require('./abi').abi;

const PROVIDER = new HDWalletProvider(
  config.MNEMONIC,
  "https://ropsten.infura.io/v3/" + config.ACCESS_TOKEN
);

const web3 = new Web3(PROVIDER);

const contract = new web3.eth.Contract(ABI, config.CONTRACT_ADDRESS);

function thanksMessage(address) {
  return contract.methods.thanksMessage(address).call();
}

function thanks(address, message) {
  return contract.methods.thanks(address, message);
}

function balanceOf(address) {
  return contract.methods.balanceOf(address).call();
}

module.exports = { thanksMessage, thanks, balanceOf };
```

ERC20トークンのコントラクトで定義したメソッドを呼び出す関数を定義している。



`abi.js`

```javascript
exports.abi = [
  {
    "constant": true,
    "inputs": [],
    "name": "name",
    "outputs": [
      {
        "name": "",
        "type": "string"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_spender",
        "type": "address"
      },
      {
        "name": "_value",
        "type": "uint256"
      }
    ],
    "name": "approve",
    "outputs": [
      {
        "name": "",
        "type": "bool"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [],
    "name": "totalSupply",
    "outputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_from",
        "type": "address"
      },
      {
        "name": "_to",
        "type": "address"
      },
      {
        "name": "_value",
        "type": "uint256"
      }
    ],
    "name": "transferFrom",
    "outputs": [
      {
        "name": "",
        "type": "bool"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [],
    "name": "decimals",
    "outputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_spender",
        "type": "address"
      },
      {
        "name": "_subtractedValue",
        "type": "uint256"
      }
    ],
    "name": "decreaseApproval",
    "outputs": [
      {
        "name": "",
        "type": "bool"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [
      {
        "name": "_owner",
        "type": "address"
      }
    ],
    "name": "balanceOf",
    "outputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [],
    "name": "symbol",
    "outputs": [
      {
        "name": "",
        "type": "string"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_to",
        "type": "address"
      },
      {
        "name": "_value",
        "type": "uint256"
      }
    ],
    "name": "transfer",
    "outputs": [
      {
        "name": "",
        "type": "bool"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_spender",
        "type": "address"
      },
      {
        "name": "_addedValue",
        "type": "uint256"
      }
    ],
    "name": "increaseApproval",
    "outputs": [
      {
        "name": "",
        "type": "bool"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [
      {
        "name": "_owner",
        "type": "address"
      },
      {
        "name": "_spender",
        "type": "address"
      }
    ],
    "name": "allowance",
    "outputs": [
      {
        "name": "",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "name": "_initialSupply",
        "type": "uint256"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "constructor"
  },
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "name": "owner",
        "type": "address"
      },
      {
        "indexed": true,
        "name": "spender",
        "type": "address"
      },
      {
        "indexed": false,
        "name": "value",
        "type": "uint256"
      }
    ],
    "name": "Approval",
    "type": "event"
  },
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "name": "from",
        "type": "address"
      },
      {
        "indexed": true,
        "name": "to",
        "type": "address"
      },
      {
        "indexed": false,
        "name": "value",
        "type": "uint256"
      }
    ],
    "name": "Transfer",
    "type": "event"
  },
  {
    "constant": false,
    "inputs": [
      {
        "name": "_to",
        "type": "address"
      },
      {
        "name": "_message",
        "type": "string"
      }
    ],
    "name": "thanks",
    "outputs": [],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "constant": true,
    "inputs": [
      {
        "name": "_address",
        "type": "address"
      }
    ],
    "name": "thanksMessage",
    "outputs": [
      {
        "name": "",
        "type": "string"
      }
    ],
    "payable": false,
    "stateMutability": "view",
    "type": "function"
  }
];
```

このABIはコントラクトのコンパイルで出来たビルドファイルの中からコピペして作成している。



`index.js`

```javascript
const Botkit = require('botkit');
const config = require('./config');
const contract = require('./contract');
const math = require('./utils/math');

const controller = Botkit.slackbot({
  debug: false,
  json_file_store: './storage'
});

controller.spawn({
　　token: config.BOT_TOKEN,
}).startRTM(function(err, bot, payload) {
　　if (err) {
　　　　console.error('Error: Could not connect to Slack');
　　　　process.exit(1);
　　}

  /**
   * Welcome!!
   */
  controller.hears(['welcome'], 'direct_message, direct_mention, mention', (bot, message) => {
    bot.reply(message, `\`\`\`
こんにちは!!社内トークンボットβ版です。
このボットから INACOIN（INA）トークンを送受信することができます。
使い方はこのボットにメンション or DM で \`help\` で確認することができます。
\`\`\`
    `);
  });

  /**
   * 使い方
   * help
   */
  controller.hears(['help'], 'direct_message, direct_mention, mention', (bot, message) => {
    bot.reply(message, `
- address の登録（※ ropsten の address 登録してください）
\`\`\`
add {address} 
\`\`\`
- token とメッセージを送る
  *メッセージはブロックチェーン上に永遠に記録されます！*
\`\`\`
send \`{message}\` to @{mention}
\`\`\`
- 自分宛てに届いた最新のメッセージを見る
\`\`\`
show message
\`\`\`
- 残高確認
\`\`\`
balance
\`\`\`
- このボットの説明
\`\`\`
welcome
\`\`\`
    `);
  });

  /**
   * ropsten アドレス登録
   * `add ${address}` で address を登録する
   */
  controller.hears(['add ([a-zA-Z0-9]+)$'], 'direct_message, direct_mention, mention', (bot, message) => {
    const address = message.match[1];
    console.log('address: ', address);
    controller.storage.users.get(message.user, (getStorageErr, savedUserInfo) => {
      if (getStorageErr) {
        console.error('getStorageErr: ', getStorageErr);
      }
      const userInfo = {
        id: message.user,
        address,
      };
      console.log('userInfo: ', userInfo);

      controller.storage.users.save(userInfo, (postStorageErr, id) => {
        if (postStorageErr) {
          console.error('postStorageErr: ', postStorageErr);
        }
        bot.reply(message, `<@${id}> あなたのアドレスは *${userInfo.address}* ですね。登録しました！`);
      });

    });
  });

  /**
   * 自分宛てに届いた最新のメッセージを見る
   * `show message`
   */
  controller.hears(['show message'], 'direct_message, direct_mention, mention', (bot, message) => {
    controller.storage.users.get(message.user, (getStorageErr, savedUserInfo) => {
      if (!savedUserInfo) {
        bot.reply(message, `<@${message.user}> まず、 \`add {address}\` で wallet（ropsten）アドレスを登録してください。`);
        return;
      }
      // storage に address があればメッセージを返す
      contract.thanksMessage(savedUserInfo.address)
        .then((res) => {
          console.log('thanksMessage: ', res);
          bot.reply(message, `<@${message.user}> 直近のあなたへのメッセージ: \`${res}\``);
        });
    });
  });

  /**
   * token（とメッセージ）を送る
   * `send `${message}` to @${mention}`
   */
  controller.hears(['send `(.*)` to <@(.*)>$'], 'direct_message, direct_mention, mention', (bot, message) => {
    const sendTo = message.match[2];
    const sendMsg = message.match[1];

    controller.storage.users.get(sendTo, (getStorageErr, savedUserInfo) => {
      if (!savedUserInfo) {
        bot.reply(message, `<@${message.user}> そのユーザーのアドレスは登録されていません。`);
        return;
      }
      // TODO: 自分のアカウントには送れないようにする
      // storage に address があればメッセージを返す
      console.log(`${savedUserInfo.address} に ${sendMsg} を送る`);
      contract.thanks(savedUserInfo.address, sendMsg)
        .send({
          from: config.PARENT_ADDRESS,
        })
        .on('transactionHash', () => {
          bot.reply(message, '送信中です…');
        })
        .on('confirmation', (confirmationNumber, receipt) => {
          // 最初の1回の承認で終わりにする
          if (confirmationNumber < 1) {
            bot.reply(message, `<@${message.user}> ユーザー（<@${sendTo}>）に token と メッセージ を送りました！`);
          }
        })
        .on('error', (err) => {
          console.error('thanks error: ', err);
          bot.reply(message, `<@${message.user}> token 付与に失敗しました。`);
        });

    });
  });

  /**
   * 残高確認
   * `balance`
   */
  controller.hears(['balance'], 'direct_message, direct_mention, mention', (bot, message) => {
    controller.storage.users.get(message.user, (getStorageErr, savedUserInfo) => {
      if (!savedUserInfo) {
        bot.reply(message, `<@${message.user}> まず、 \`add {address}\` で wallet（ropsten）アドレスを登録してください。`);
        return;
      }
      contract.balanceOf(savedUserInfo.address)
        .then((res) => {
          const balance = `${math.toLocaleString(res / 1e18)} ${config.SYMBOL}`;
          console.log('balance: ', balance);
          bot.reply(message, balance);
        });
    });
  });

  /**
   * storage の中身を全部表示する
   * `get storage`
   * * debug 用 *
   */
  controller.hears(['get storage'], 'direct_message, direct_mention, mention', (bot, message) => {
    controller.storage.users.all((err, allUserData) => {
      console.log('allUserData', allUserData);
      bot.reply(message, JSON.stringify(allUserData));
    });
  });

  /**
   * storage に直接データを入れる
   * `set storage ${JSON}`
   * * debug 用 *
   */
  controller.hears(['set storage `(.*)`'], 'direct_message, direct_mention, mention', (bot, message) => {
    const string = message.match[1];
    try {
      const json = JSON.parse(string);
      controller.storage.users.save(json, (err) => {
        if (err) {
          bot.reply(message, '登録失敗しました。');
          return;
        }
        bot.reply(message, '登録しました。');
      });
    } catch (e) {
      bot.reply(message, '登録失敗しました。');
    }
  });

});
```

実際にbotとしてのやりとりを記述している部分。



# License

MIT

