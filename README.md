##### 0. 前提

1. ASで用意されている、Google Maps Activity templateを使用したサンプル
    1. google_maps_api.xml
        1. 設定ファイル
        2. これもdebug用とrelease用がある
    2. activity_maps.xml
    3. MapsActivity.java
2. google map用語
    1. type
        1. normal
        2. hybrid
        3. satelite
        4. terrain
    1. markers
        1. 位置を指し示すアイコン。赤いやつ
    1. info window
        1. markerに追加できるポップアップ
    1.  points of interest(POIs)
        1. 直訳は観光スポット
        1. 公園や学校、自治体デフォルトで表示される(regular poi)
        2. business poiは商業施設。区別される
            1. normalだと、regular poiに加えてこれらも表示される
        1. info windowはデフォルトでは表示されない
    1. pan
        1. カメラのpanのこと。水平方向や垂直方向に振る撮影方法
        2. map上を移動すること
        3. **google mapでは画面をカメラのファインダーに見立てて、mapを撮影しているイメージ**
    1. zoom
        1. levelがある
    1. overlay
        1. ground overlay
            1. map上のある地点に固定される画像レイヤー
            1. 地図上に画像を追加する際などに便利
        2. tile overlay

##### 1. API keyをセットする
1.  API keyを取得
2. google_maps_api.xmlにセットする。
    1. Manifestではない

##### 2. Add map types and markers
0. 前提
    1. app barにoptions menuを用意
    2. そこで、typeを変えられるようにする
1. 手順
    1. options menu用のlayoutを用意する
    2. app barを使うためにFragmentActivityからAppCompatActivityのサブクラスに変更する
    3. onCreateOptionsMenu()を実装する
    4. イベントリスナーonOptionsItemSelected()でoptionごとに、 setMapType() でmapのtypeを設定する

##### 3. Move the default map location
1. 手順
    1. onMapReadyメソッドでLatLngオブジェクトを作成する
    2. zoomレベルをfloat変数で設定する
    3. LatLngとzoomを渡してCameraUpdateオブジェクトを作成
    4. moveCameraでカメラを動かす

##### 4. Add map markers
1. 手順
    1. GoogleMapオブジェクトにclicklistenerをセットするためのメソッドを用意する
    2. その中でaddMarker()でmarkerを追加する
    3. onMapReadyで1.のメソッドを呼ぶ

##### 5.  Add POI listener
0. 前提
    1. poiはデフォルではinfo windowが表示されない
    2. なので、表示するようにする
    3. 実際はpoi自体からinfo windowを表示するというよりは、選択したpoiの情報を持ったmarkerを作りなおして、そのmarkerからinfowindowを表示させるようにする
1. 手順
    1. GoogleMapオブジェクトにpoiclicklistenerをセットするためのメソッドを用意する
    2. その中でpoiの位置情報を持ったmarkerを作る
    3. そのmarkerでinfowindowを表示するようにする
    4. onMapReady()で1.のメソッドを呼ぶ

##### 6. Style your map
0. 前提
    1. google mapはスタイルをカスタマイズできる
        1. map自体
        2. marker
        3. 画像などの層をgoogle mapの上に乗せれる(overlay)
            1. 今回はground overlayで画像を表示する
1. map自体
    1. 以下のサイトでスタイルを作成し、jsonでエクスポートする
        1. [mapstyle](https://mapstyle.withgoogle.com/)
    1. resディレクトリ配下にrawディレクトリ作成（Android resourse directory）
    2. raw配下にmap_style.jsonを作成し、エクスポート内容をペーストする
    3. onMapReady内でsetMapStyleでセットする
1. marker
    1.  MarkerOptions()でセットする
1. overlay
    1. onMapReadyメソッド内で、カメラ移動後に GroundOverlayOptionsオブジェクトを作成する
        1. ここで表示する画像を食わせる
        2. そのあとに表示位置をセットする
    1.  GoogleMapオブジェクトでaddGroundOverlay()を呼ぶ

##### 7. Enable location tracking and Street View
0.  前提
    1. location-data layer を使って現在地を表示する
        1. Location APIをわざわざ使わなくてもいい
        2. location buttonも自動的に表示される
    2. Street Viewは、今回はPOIのinfo windowをタップした時に表示されるように実装する
        1. まず、makerをpoi makerとそれ以外のmakerと区別できるようにする
            1. コード上では、poiも単なるmarkerオブジェクト
        2. poiのinfo windowをタップされたら、現在のMapFragmentからStreetViewPanoramaFragmentに張り替える
1. location tracking
    1. Manifestに ```FINE_LOCATION``` permissionが設定されていることを確認する
    2. permissionの確認をするメソッドを作る [**the runtime-permission model**]
        1. locationの時と同じ
        2. permissionがあれば、setMyLocationEnabled()でlocation layerを有効にする
        2. permissionがなければ、  ActivityCompat.requestPermissions()でユーザーにpermissionを求める
    3. permission requestのイベントハンドラで、permissionが与えられたらまた 2.のメソッドを呼ぶようにする
2. Street View
    1. onPoiClick内で、作成したpoi makerオブジェクトにtagをセットする
    2. fragmentをホストするために、activity_maps.xmlをFrameLayoutに変更する
    3. onCreate上で、IDでfragmentをinflateするのをやめる。2.で静的なfragmentがなくなっているので
    4. その代わりに動的にfragmentを作成する。
    5. そして、そのfragmentオブジェクトにレイアウトを追加する
    6. 非同期でmap loadはそのまま
    7. OnInfoWindowClickListener()をセットするメソッドを用意する
    8. イベントハンドラーの中で、makerオブジェクトのtagを確認する
    9. tagを持っているmaker(つまりpoi) だったらmarkerの位置からStreetView用の位置optionを作成する
    10. そのoptionを使ってStreerViewPanoramaFragmetnオブジェクトを作る
    11. Fragmentのトランザクション
        1. Fragmentをreplaceする
        2. backボタンを押した時にアプリから出ないようにback stackに追加する。これで、元のSupportMapFragment戻れる
        3. addToBackStack()の引数は、backstackを操作する際に使われる名前らしいが、今回は使うことはないのでnull
    12. onMapReadyで上で作成したメソッドを呼ぶ。（setPoiClick()のあとに配置）
