# さかひろ備忘録
## 1.セットアップ関連

### 1-1.加藤研究室に入ってまず入れたもの

| 名前  |  リンク  |　備考 |
| ---- | ---- | ---- |
|  VScode  |  [リンク](https://code.visualstudio.com/)  | Anacondaやgitと連携しておくと便利 |
|  Anaconda  |  [リンク](https://www.anaconda.com/products/individual)  | パスを通すこと |
| slack | [リンク](https://slack.com/intl/ja-jp/downloads/windows) | |
| git | [リンク](https://git-scm.com/) | 全てnextで可 |
| その他 | | gmailのcv-allに入れてもらう |

### 1-2.VScodeでanacondaを使う

1. VScodeのメニューバーから`ファイル`>`ユーザー設定`>`設定`
2. 検索で`Python conda`で検索すると`Python:Conda Path`が出てくる
3. ここにパスを入力 例)`C:\Users\Hiroki Sakai\anaconda3\python.exe`
4. 実行したいPythonファイルを開き右下の`Python 3.9.7`を押す
5. 実行したい環境に変える

  [参考になるサイト](https://blog.beachside.dev/entry/2017/12/25/000000)


### 1-3.Dockerのsetupについて

1. [git](https://git-scm.com/)のインストール。すべてnextで可
2. 稲垣先輩のデータをもとに`dockerfile`を作る
3. `Windows PowerShell`を起動
4. ファイルを作る。名前は何でも可。
```
mkdir docker_setup_dir
```
5. ファイルに移動
```
cd .\docker_setup_dir
```
6. 稲垣先輩のファイルをクローン
```
git clone http://10.226.47.83:8080/Inagaki/docker_setup_sample.git
```
7. `git`の登録する`名前`と`email`の変更
```
git config --global user.email “sakai@cv.info.gifu-u.ac.jp”
git config --global user.name “hiroki sakai”
```
8. ファイルに移動
```
cd .\docker_setup_sample
```
9. `origin`の位置に設定
```
git remote set-url origin http://10.226.47.83:8080/sakai/sakai_docker_setup.git
```
10. `VScode`を現ファイルで開く
```
code ./
```
11. `Vscode`上で`README.md`に従う(12~16)
12. `docker-compose.yml`：`hostname`, `container_name`, `image`, `ports`を適宜変更する
13. `build/Dockerfile`：1行目を使用したい`cuda`バージョンに合わせて変更  [参考](https://hub.docker.com/r/nvidia/cuda/tags?page=1&ordering=last_updated) 
14. `exec.sh`：`docker`コンテナの`root`ユーザにログインする`shell script`。にコンテナ名を入力する。
15. `build/config/envs/develop.yml`：`conda`の環境構築用ymlファイル。`conda env export -n [yout env name] >> [filename].yml`で作成できる。自分の好きな環境を構築しよう。
16. 必要に応じて自分に割り当てられたポート番号を使用する。 ポート番号の割り当ては以下を[参照](https://docs.google.com/spreadsheets/d/1lTFTDwoAf_W3r1QtNZFpbSBec4tG5pjG_IoWEcKD4mY/edit?usp=sharing)

例：54000-54999が割り当てられている場合のポートフォワーディング
```
[ssh] 54022:22
[jupyter] 54888:8888
[tensorboard] 54006:6006
```

Docker-compose.yml完成図(例)

```
version: '2.3'
services:
  python_dev_env:
    build: ./build
    image: sakai_cuda:11.0-runtime-ubuntu18.04
    container_name: sakai_container
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
    ports:
      - "34022:22"
      - "34888:8888"
      - "34006:6006"
    volumes:
        - /mnt/qnap2:/home/
    restart: always
    hostname: "sakai"
    shm_size: '4gb'
```

18. `Dockerfile`の80,81行目をコメントアウト
19. `git`に`push`する
```
git add .
git commit –m “何でもいい”
git push origin master
```
20. この時点で次のコードでGPUに入れる
```
ssh sakai@10.226.47.35 -p 34022
```
21. 簡単化したいなら、`Users/Sakai/.ssh/`に`config`を作り以下を記入

```
Host gpu*
    User sakai
Host gpu25
    HostName 10.226.47.35
    port 34022
```

22. `git`に`push`する
```
git add .
git commit –m “何でもいい”
git push origin master
```
23. 21を行った場合以下だけで入れるようになる
```
ssh gpu25
```


### 1-4.GPUにcondaの環境を作る
1. コピーしたい`conda`の環境の名前を入れて

```
conda env export –n [env_name] > env_name.yml
```

2. `yml`ファイル内の2個目の=以降を消す 
```
例>_pytorch_select=0.1
```
3. 上記で作った`yml`ファイルをエクスプローラーで `\\10.226.47.81\home`　に移動
4. `Windows PowerShell`などでGPUに入ったら以下を実行
```
conda init
cat .bashrc　
```
5. 以下で`anaconda`に入る(おそらく(base)と左に表示されるようになるはず)
```
bash
```
6. `conda`の環境を構築(もしエラーが出たらそのライブラリはymlファイルから消す)
```
conda env create –n 環境名 –f ymlファイル
```

### 1-5.VPN接続の仕方

| 呼称  |  説明  |
| ---- | ---- |
|  ローカル  | 加藤研究室にあるPCのこと  |
|  GPU  |  GPUのこと(どの番号でもよい)  |
| GPU4 | GPU4@10.226.47.14のこと |
| 自宅PC | 自宅で使うPCのこと |

1.　ローカルからroot権限でGPUに入る
```
ssh gpu25@10.226.47.35
```
2. 以下で`.ssh`フォルダの操作
```
//　.sshフォルダがあるかを確認
ls -a

// なかったら
mkdir .ssh

//あったり作ったりしたら入る
cd .ssh

//以下でカギを作成
ssh-keygen -t rsa -b 4096

//鍵が生成されたのを確認
ls -a

//公開鍵をコピーする
cat id_rsa.pub >> authorized_keys

//権限をそれぞれ変える
chmod 600 authorized_keys
chmod 700 ~/.ssh

//一応authorized_keysがあるかを確認してから公開鍵を消す
rm id_rsa.pub
```

3. ローカルのGUIでid_rsaをローカルの`.ssh`に転送
4. 以下でGPUから別のGPUに接続できるのかを確認
```
ssh gpu25@10.226.47.35
ssh -i id_rsa gpu40@10.226.47.50
```

5. `.ssh`内のconfigに以下のように記入
```
Host gpu*
    User sakai
    IdentityFile C:\Users\sakai\.ssh\id_rsa

Host gpu4
    HostName 10.226.47.14
    port 2222
```
6. GPU4に入れることを確認
```
ssh gpu4
```
7. 自宅PCの`.ssh`にid_rsaとconfigを送る
8. 自宅PC上でconfigのIdentityFileを自宅PCのid_rsaのパスにし、gpu4のHostnameを`133.66.226.47`にする
9. 自宅PC上で[このリンク]()からVPN接続をする(ただし、岐阜大学のWiFiにつないでいると接続できない)
10. GPU4に自宅PCからつなげるかを確認
```
//GPU4に入る
ssh gpu4

//GPU4に入れたらそこからQnapに接続
ssh sakai@10.226.47.35 -p 34022

//確認するとファイルがあるはず
ls
```

[参考にしたサイト①](https://qiita.com/nnahito/items/dbe6fbfe347cd66ae7e6)

[参考にしたサイト②](https://qiita.com/kazokmr/items/754169cfa996b24fcbf5)

[VPN接続の文書]()

## 2.結果の保存と自動化

### 2-1.Tmuxについて

Tmux上で動かさないとスリープすると落ちる。

操作一覧
```
tmux 	new –s セッション名     セッションを立ち上げる 
Ctrl + b -> d                 セッション離脱（プログラムは動き続ける）
tmux a				                直近のセッションに戻る
tmux -t セッション名          セッションに戻る
```

[チートシート](https://qiita.com/nmrmsys/items/03f97f5eabec18a3a18b)

### 2-2.Argparseでymlファイルから参照する

使うときには`import yaml`及び`import argparse`でインポート

Argparseでコマンドライン引数を指定する
```
parser = argparse.ArgumentParser(description='YAML example')
parser.add_argument('--optim', type=str, required=True, help='optimizer name')
parser.add_argument('--lr', type=float, required=True, help='learning rate')
args = parser.parse_args()
```

コマンドラインでyamlを指定して読みだす(train.py)
```
def get_args():
    parser = argparse.ArgumentParser(description='YAMLありの例')
    parser.add_argument('config_path',type=str,help='設定ファイル(.yaml)')
    args = parser.parse_args()
    return args

def make_optimizer(params,name,**kwargs):
    return optim.__dict__[name](params,**kwargs)

with open(get_args().config_path, 'r') as f:
    config = yaml.safe_load(f)

epochs = config['epochs']

```

yamlの例
```
batchsize: 32
configname: '320.116'
epochs: 100
optimizer:
  lr: 0.1
  name: AdamW
  weight_decay: 0
resize_size: 16
```

yamlを作るスクリプト(make_config.py)
```
batch_list = [32,128]
lr=[0.1,0.01,0.001]
resize_list=[16,32,64]

for b in batch_list:
    for l in lr:
        for r in resize_list:
            with open("config/"+str(b)+str(l)+str(r)+".yaml", "wt") as yf:
                        yaml.dump({
        "configname": str(b)+str(l)+str(r),
        "optimizer":
            {"name": "AdamW",
            "lr": l,               #途中で変えるといい　大→小　スケジューラ
            "weight_decay":0,},
        "batchsize":b,
        "resize_size":r,
        "epochs": 100
    }, yf, default_flow_style=False)
yf.close
```

[参考にしたサイト](https://rightcode.co.jp/blog/information-technology/pytorch-yaml-optimizer-parameter-management-simple-method-preparation)

### 2-3.Loggingでテキストを残す

使うときには`import logging`でインポート

logの残し方
```
#設定
logging.basicConfig(level =logging.DEBUG, filename=path_output+'/output.text' ,filemode='w',format="%(asctime)s %(message)s")

#ログを残す
logging.debug(config)
```

[参考にしたサイト](https://symfoware.blog.fc2.com/blog-entry-883.html)

### 2-4.TensorBoardについて

使うときは`from torch.utils.tensorboard import SummaryWriter`でインポート

結果を残す
```
#writerの初期化
writer = SummaryWriter(log_dir="./logs/"+config['configname'])

#結果を残す
writer.add_scalars("loss",{"train_loss": train_avg_loss,"valid_loss": valid_avg_loss},epoch)

#writerを閉じる
writer.close
```

[参考にしたサイト①](https://torch.classcat.com/2020/05/14/pytorch-1-5-recipes-tensorboard-with-pytorch/)

[参考にしたサイト②](https://www.oio-blog.com/contents/tensorboard)

### 2-5.Graphviz→pytorchviz(使わなかった)

1. [AnacondaにGraphVizを導入](https://qiita.com/pyg50/items/eeae7c51e23d8a036bf1)
2. [PytorchvizをGithubから導入](https://github.com/szagoruyko/pytorchviz)

[実際に使っているサイト](https://gurutaka-log.com/pytorch-torchviz-visualize)

### 2-6.シェルスクリプトを使ったMulti Runのやり方

`ShellScript`の実行は`bash ~~.sh`

使っている例(main.sh)
```
files= $PATH + "/config/"
for file in `\find $files -name '*.yaml'`; do
    #echo $file
    python train.py $file
    wait
done
```

[参考にしたサイト①](https://qiita.com/suzuki_y/items/52160149baae1ac3d73c)

[参考にしたサイト②](https://masaeng.hatenablog.com/entry/2019/08/13/232015)

### 2-7.スクリプト上でディレクトリなければ作って実行

使うときは`import os`でインポート

使っている例(train.py)
```
if os.path.exists(path+config['configname']+str(num)) ==False:
    os.makedirs(path+config['configname']+str(num))
```

[参考にしたサイト](https://note.com/yokkai/n/nc50c5860d914)

### 2-8.Matplotlibの図を保存

使うときは`import matplotlib as plot`でインポート

使っている例
```
#saveする
plot.savefig(“図の名前”)

#初期化する
plot.clf()
```

一連の流れ(train.py)
```
# Lossを描画
plt.clf()
plt.plot(history['train_loss'])
plt.plot(history['valid_loss'])
plt.title('Model loss')
plt.ylabel('Loss')
plt.xlabel('Epoch')
plt.legend(['train loss', 'validation loss'], loc='upper right')
plt.grid(True)
plt.show()
plt.savefig("TrainOutput/Loss.png")
```

ただし、書いたグラフや点が残ったままになってしまうので毎度初期化をすること

[参考にしたサイト](https://techacademy.jp/magazine/22285)

## 3.チューニング発表1/12

### 3-1.参考になった論文

[深層学習を用いた画像識別タスクの精度向上テクニック](https://www.guruguru.science/competitions/17/discussions/4864ee81-6336-4cad-bb74-a4c9b46e6eb2/)

### 3-2.Schedulerで学習率を変える

使うときは`import torch.optim as optim`でインポート

Schedulerの使い方
```
#宣言
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=20, gamma=0.5)

#時間を進める
scheduler.step()
```

[参考にしたサイト](https://wonderfuru.com/scheduler/)

STEPLR以外については[こちら](https://katsura-jp.hatenablog.com/entry/2019/01/30/183501)

ただし表示はscheduler.get_last_lr()[0]がよい

### 3-2.過学習対策

[過学習について①](https://data-viz-lab.com/overfitting)

[過学習について②](https://www.tensorflow.org/tutorials/keras/overfit_and_underfit?hl=ja)

対策1. [L1/L2正則化](https://qiita.com/tabintone/items/790729a89ed84bb21b74)

対策2. [ドロップアウト](https://sonickun.hatenablog.com/entry/2016/07/18/191656)

### 3-3.最適化関数について

使うときは`import torch.optim as optim`でインポート

最適化関数の使い方
```
#SGD
optim.SGD(params, lr=<required parameter>, momentum=0, dampening=0, weight_decay=0, nesterov=False)

#Adam
optim.Adam(params, lr=0.001, betas=(0.9, 0.999), eps=1e-08, weight_decay=0, amsgrad=False)

#AdamW(weight_decayの影響の大きいAdam)
optim.AdamW(params, lr=0.001, betas=(0.9, 0.999), eps=1e-08, weight_decay=0, amsgrad=False)
```

今回はyamlから構築(train.py)
```
def make_optimizer(params,name,**kwargs):
    return optim.__dict__[name](params,**kwargs)

#宣言
optimizer = make_optimizer(model.parameters(),**config['optimizer'])
#optimizer = optim.Adam(model.parameters(), lr=0.01)  # 最適化関数Adam
```

[参考にしたサイト](https://rightcode.co.jp/blog/information-technology/torch-optim-optimizer-compare-and-verify-update-process-and-performance-of-optimization-methods)

### 3-4.cifer10で使えるModel

この[サイト](https://github.com/kuangliu/pytorch-cifar)から拝借

違うファイルのclassを使う方法
```
#import
from dir.file import TmpName

#初期化
a = TmpName()
```

[実装方法](https://testpy.hatenablog.com/entry/2020/01/12/171347)

### 3-5.TTAの導入

[TTAch](https://github.com/qubvel/ttach#transforms)の実装

[使っている例](https://qiita.com/cfiken/items/7cbf63357c7374f43372https://www.guruguru.science/competitions/17/discussions/4864ee81-6336-4cad-bb74-a4c9b46e6eb2/)

### 3-6.正則化について

[参考にしたサイト](https://qiita.com/cfiken/items/7cbf63357c7374f43372)

[最適化関数と正則化](https://acro-engineer.hatenablog.com/entry/2019/12/25/130000#AdamW)

### 3-7.データ拡張について

[参考にしたサイト](https://pystyle.info/pytorch-list-of-transforms/)

[精度比較ありのサイト](https://qiita.com/enbi/items/f84d253b79184c903c27)

## 4.最終発表3/17

### 4-1.LynuxのUSB Boot

[参考にしたサイト](https://demura.net/education/lecture/20509.html)

USBとSSD両方が必要！！

### 4-2.CoppeliaSimのインストール

[リンク](https://coppeliarobotics.com/downloads) - Edu版にすること

### 4-3.CoppeliaSimのチュートリアル

[チュートリアル](https://www.coppeliarobotics.com/helpFiles/index.html)

特に[BubbleRob](https://www.coppeliarobotics.com/helpFiles/en/bubbleRobTutorial.htm)は操作をほぼすべて行うのでやること

### 4-4.CoppeliaSimのRemoteAPI

Pythonのスクリプトを動かすための

[RemoteAPIについて①](https://testpy.hatenablog.com/entry/2017/09/18/005847)

[RemoteAPIについて②](https://chachay.hatenablog.com/entry/2016/05/05/172415)

ただし、ポート番号は20000以上が吉

遅いので後述のPyRepで行ったほうが良い

### 4-5.Pygameについて

コントローラーの入力を得るため[Pygame](https://qiita.com/suo-takefumi/items/c9c4b23ea74a56d7e901)を使う

ただし、インストールはpipを使った(あんまりよくない)

### 4-6.タイヤの運動方程式(対軸2輪ロボット)

[参考にしたサイト](https://www.mech.tohoku-gakuin.ac.jp/rde/contents/course/robotics/wheelrobot.html)

各パラメータまとめ

- ρ:𝜐によって決まる
- 𝑤:コントローラの入力で決まる
- v:自分で決める
- d:ロボットによって異なる

### 4-7.PyRepについて

[PyRep](https://github.com/stepjam/PyRep)

[PyRepの各種関数](https://pyrep.readthedocs.io/en/latest/)

### 4-8.OpenCV

OpenCVのインストール
```
pip install opencv-python opencv-contrib-python
```

OpenCVとPyRepは同時にはうまく動かないので、別の仮想環境にUDPで画像を飛ばしてから使うこと

### 4-9.UDPについて

1. Sender側
   -pip install opencv-python-headless opencv-contrib-python
2. Reciever側
   -pip install opencv-python opencv-contrib-python
