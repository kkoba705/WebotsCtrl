ロボットシミュレータ Webots (https://cyberbotics.com/) のコントローラを C++ で作成する際に CMake の find_package() で必要なライブラリを利用できるようにします。自作のコントローラにさまざまなライブラリをリンクして開発するのが容易になります。

Visual Studio Code に CMake の拡張機能を入れておくと、ソース入力時のコード補完とエラー検出、実行ファイルのビルド、デバッグ実行までできて便利です。

# install (Windows)

適当な作業用ディレクトリから Developer Command Prompt (Visual Studio 2022) で
```
> git clone https://github.com/kkoba705/WebotsCtrl.git
> cd WebotsCtrl
> md build
> cd build
> cmake -DCMAKE_INSTALL_PREFIX=c:/dev/WebotsCtrl ..
> cmake --build . --config Debug --target install
> cmake --build . --config Release --target install
```
として ```c:\dev\WebotsCtrl``` にインストールできる。（他の場所に変更しても良い）

環境変数 ```CMAKE_PREFIX_PATH``` もしくは  ```WebotsCtrl_DIR``` に ```c:\dev``` を設定する。


# install (Ubuntu)
```~/.bashrc``` の最後に以下の内容を追加する。
```
export CMAKE_PREFIX_PATH=$HOME/dev:$CMAKE_PREFIX_PATH
export WEBOTS_HOME=/usr/local/webots
```
- ```CMAKE_PREFIX_PATH``` を設定する代わりに ```WebotsCtrl_DIR``` に ```$HOME/dev``` を設定しても良い。
- Webots をユーザー権限でインストールした場合には、```WEBOTS_HOME``` をインストール先に合わせて正しく設定する。
- 設定を反映させるために、端末（Terminal）を一旦閉じて開き直す。

適当な作業用ディレクトリから端末（Terminal）で
```
$ git clone https://github.com/kkoba705/WebotsCtrl.git
$ cd WebotsCtrl
$ mkdir build
$ cd build
$ cmake -DCMAKE_INSTALL_PREFIX=$HOME/dev/WebotsCtrl ..
$ make install
```

として ```~/dev/WebotsCtrl``` にインストールできる。（環境変数を変更して、他の場所にしても良い）

# コントローラの要件
Webots User Guide の [The "controllers" Directory](https://cyberbotics.com/doc/guide/the-standard-file-hierarchy-of-a-project#the-controllers-directory) にあるように、たとえば "sample_controller" という名前のコントローラを作成するには、Webots のプロジェクトディレクトリの直下にある "controllers" というディレクトリ内に "sample_controller" というディレクトリを作成して、そこに実行ファイル "sample_controller(.exe)" を用意すればよい。

Webots のメインメニューからたどって "New Robot Controller..." を選び、コントローラ名として "sample_controller" と入力すると、 "controllers" の下に "sample_controller" というディレクトリが作成され、そこに "sample_controller.cpp" という雛形ファイル（何もしないコントローラのソース）が用意される。（同時に "Makefile" もしくは "sample_controller.sln" も作成される。）

# コントローラーの作成方法
コントローラのソースが一つのファイル "sample_controller.cpp" だけで構成される場合で使い方を説明する。"sample_controller" ディレクトリに、以下の内容で ```CMakeLists.txt``` を作成する。
```cmake
cmake_minimum_required(VERSION 3.15)

# The project name is defined to be the controller directory name.
get_filename_component(PROJECT ${CMAKE_SOURCE_DIR} NAME)
project(${PROJECT} LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

find_package(WebotsCtrl REQUIRED)

add_executable(${PROJECT})
target_link_libraries(${PROJECT} WebotsCtrl::CppController)

target_sources(${PROJECT} PRIVATE
  ${PROJECT}.cpp
)

# Copy the target executable at the right location.
add_custom_command(TARGET ${PROJECT} POST_BUILD COMMAND ${CMAKE_COMMAND} -E
  copy $<TARGET_FILE:${PROJECT}> ${CMAKE_SOURCE_DIR}
)

# Specify the default startup project for Visual Studio
set_property(DIRECTORY ${CMAKE_SOURCE_DIR} 
  PROPERTY VS_STARTUP_PROJECT ${PROJECT})
```
* プロジェクト名とソースファイル名には、CMakeLists.txt の存在するディレクトリ名  "sample_controller" ( = ${PROJECT}) が使われる。
* 複数のソースファイルや追加のヘッダファイルを使いたいときには、target_sources() 内にファイル名を追加する。もしくはサブディレクトリを作って add_subdirectory() を使う。
* find_package() で他のライブラリを探して、target_link_libraries() でプロジェクトに追加できる。
* ```set(CMAKE_CXX_STANDARD 17)``` と変更すれば C++17（同様にしてそれ以上）も使える。

"sample_controller" ディレクトリにおいて
```
> mkdir build
> cd build
> cmake ..
> cmake --build . --config Release
```
とすることでコントローラーをビルドできる。

add_custom_command() の設定によって、実行ファイル "sample_controller(.exe)" が作成されると、それは "sample_controller" ディレクトリへとコピーされる。既存の "sample_controller(.exe)" が実行中の場合、上書きに失敗するので、コントローラのビルドはシミュレーションが停止している状態で行うよう注意する。

* Visual Studio Code を使う場合には、CMake の拡張機能を入れたうえで、"CMakeLists.txt" を含むディレクトリをフォルダーとして開く。
* Visual Studio の IDE を使う場合には、```> cmake ..``` で "build" ディレクトリに作成された "sample_controller.sln" を開く。

# デバッグ実行

Webots User Guide の [Running Extern Robot Controllers](https://cyberbotics.com/doc/guide/running-extern-robot-controllers) にあるように、ロボットのコントローラに `<extern>` を指定すると、コントローラを Webots の外で起動できる。これを用いて IDE からコントローラのデバッグ実行ができる。

Windows でデバッグ実行を行う場合には、環境変数 PATH に ```C:\Program Files\Webots\lib\controller``` を追加する。


# 解説記事

(Windows) https://qiita.com/kkoba775/items/d28b30a6e1f13803036f

(Ubuntu) https://qiita.com/kkoba775/items/84158c63753313c804be
