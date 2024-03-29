# いただいた質問に関する回答です（PowerShell, VBA）

PowerShell, VBA初心者ですので、間違ってたらすみません。m(__)m

## Q1について

```
Get-ChildItem | Rename-Item -NewName{ $_.Name -replace '_A([1-9])_','_A0$1_'-replace '_B([1-9])_','_B0$1_'-replace '_C([1-9])_','_C0$1_'-replace '_D([1-9])_','_D0$1_'-replace '_E([1-9])_','_E0$1_'-replace '_F([1-9])_','_F0$1_'-replace '_G([1-9])_','_G0$1_'}
```

[$_]ですが、これはパイプ変数です。  
パイプ変数とはあるメソッドの結果を次のメソッドに渡すときに使用する変数です。  
まず、Get-ChildItemでフォルダ内のファイルを集めますが、  
その結果が[$_]に入って、Rename-Itemに渡っています。  

そして、$_.Name - replace '_A([1-9]_', '_A0$1'_ で、ファイル名のA1をA01に変換しています。  
これはAの後ろが数字だったら0+数字に変換する働きをしています。

手順にするとこんな感じでしょうか。

1. Get-ChildItemで[$_]に[20220325_220311_d35_PL1_A1_4X_PH_2359.tif]が入る. 
1. $_.Name -replace '_A([1-9])_','_A0$1_'で[20220325_220311_d35_PL1_A01_4X_PH_2359.tif]ができる
1. Rename-Item -NewName で[20220325_220311_d35_PL1_A1_4X_PH_2359.tif]が[20220325_220311_d35_PL1_A01_4X_PH_2359.tif]に置き換わる

## Q2について

ファイル名の書式が変わらないのであれば、今のコマンドでも問題ないと思います。  
私も勉強がてら作ってみましたが、長くなってイマイチな気はします。何を行っているかは分かりやすいつもりですが...一応、載せておきますね。  

```
# 拡張子がtifのファイルを集めます
$filter = "*.tif"
if ($args.Length -eq 1)
{
    $filter = Join-Path $Args[0] "*.tif"
}
#Write-Output $filter
$files = Get-ChildItem -Path $filter

#!----- IDの数値を2桁に変更します -----!
$capno = 1;
foreach($file in $files)
{
    # まずファイル名をアンダーバー(_)で区切ります
    # ファイル名は"年月日_（時分秒）_???_???_ID_??_??_連番.tif"という形式と仮定します。
    $array = $file.Name.Split('_')
    #Write-Output $file.Name

    # IDは5番目なので5番目（PowerShellでは0から数えるので4になっています）の文字列を取得します
    $id = $array[4]
    #Write-Output $id

    # IDを英字＋番号に分割します
    # なお、IDは英字（1桁）＋数値（それ以外）と仮定しています。汎用性を求めるなら英字と数値の違いを調べる必要があります...
    $alpha = $ID.SubString(0, 1)
    $no = $ID.Remove(0, 1)

    # 数字を2桁にして新しい文字列を作ります。3桁にしたければPadLeft(3, '0')にすればOKです
    $newID = $alpha + $no.PadLeft(2, '0')
    #Write-Output $newID

    # 新しいファイル名を作成します
    # 以下でやっていることは $newFileName = $array[0] + $array[1] + $array[2] + $array[3] + $newID + $array[5] + $array[6] + $array[7]と同じです
    $newFileName = "";
    for($i=0; $i -lt $array.Length; $i++)
    {
        if ($i -eq 4)
        {
            $newFileName += $newID;
        }
        else
        {
            $newFileName += $array[$i]
        }

        # 最後以外は"_"を付加する
        if ($i -lt $array.Length -1)
        {
            $newFileName += "_"
        }
    }
    #Write-Output $newFileName

    #!----- キャプチャ番号を付加します ------!
    $capno_ext = "_cap" + $capno.ToString("D2") + ".tif"
    $newFileName = $newFileName.Replace(".tif", $capno_ext)
    $capno++;

    # 新しいファイル名で置き換えます
    $newFile = Join-Path $file.DirectoryName $newFileName
    Rename-Item $file $newFile
    #Write-Output $newFile
}
```

## Q3について

_cap0xを付けられるようスクリプトを修正しました。  
"キャプチャ番号を付加します"とコメントしている部分です。

## Q4について

セルを横に移動しながら背景を塗りつぶすマクロを書いてみました。  
こちらの方が分かりやすいかと思ったので。  

```
' セルを塗りつぶすマクロです（４列３行）
Sub select_celss()
    Dim rowNumber
    Dim colNumber
    Dim rowStart
    Dim colStart
    
    ' 色配列の初期化(R=3,G=4,B=5)
    Dim colorIndexes() As Variant
    colorIndexes = Array(3, 4, 5)
    
    ' セルを塗りつぶす
    colStart = 2
    rowStart = 2
    For colNumber = 0 To 2
        For rowNumber = 0 To 3
            Cells(colStart + colNumber, rowStart + rowNumber).Interior.colorIndex = colorIndexes(colNumber)
        Next rowNumber
    Next colNumber
End Sub
```

