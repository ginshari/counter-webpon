<template>
  <v-row justify="center" align="center">
    <v-col cols="12" md="8">
      <div class="text-h4">ウェブポン集計ツール</div>
      <div class="text-h5 mt-8 mb-4">概要</div>
      <div class="text-body1">
        ウェブポン(<a href="https://webpon.net/" target="_brank"
          >https://webpon.net/</a
        >)の購入明細を集計してCSVに変換するツールです
      </div>
      <div>
        バグ報告は<a href="https://twitter.com/nanasu_n" target="_brank"
          >@nanasu_n</a
        >まで
      </div>
      <div class="text-h5 mt-8 mb-4">使い方</div>
      <div class="text-body1">
        <ol>
          <li>購入明細メールの内容をコピーする</li>
          <li>入力テキストボックスに貼り付ける</li>
          <li>変換ボタンをクリックする</li>
          <li>出力テキストボックスに集計結果が表示される</li>
          <li>コピーボタンをクリックする</li>
          <li>クリップボードに出力テキストボックスの内容がコピーされる</li>
        </ol>
      </div>
    </v-col>
    <v-col cols="12" md="8">
      <v-textarea v-model="inputData" color="teal" label="入力" outlined />
    </v-col>
    <v-col cols="12" md="8">
      <v-btn block color="teal" class="mb-16" @click="convert">変換</v-btn>
    </v-col>
    <v-col cols="12" md="8">
      <v-textarea
        v-model="outputData"
        color="teal"
        readonly
        label="出力"
        outlined
      />
    </v-col>
    <v-col cols="12" md="8">
      <v-btn block color="teal" @click="copy">クリップボードにコピー</v-btn>
    </v-col>
  </v-row>
</template>

<script>
import VueClipboard from 'vue-clipboard2'

export default {
  data() {
    return {
      inputData: `[KTHR2-B01]	[1個]	スマホスタンド《A ver.》\n[KTHR2-D02]	[1個]	アクリルキーホルダー《B ver.》\n[KTHR2-D02]	[2個]	アクリルキーホルダー《B ver.》`,
      outputData: ''
    }
  },
  methods: {
    convert() {
      if (!this.inputData) {
        alert('未入力です')
      }

      try {
        // 行ごとに分割
        const inputArray = this.inputData.trim().split('\n')

        // jsonに変換
        let id = ''
        let amount = 0
        let name = ''
        const inputTable = inputArray
          .filter((row) => row.trim() !== '')
          .map((row) => {
            // 1列目(ID)
            id = row.split(']')[0].trim().replace('[', '')
            // 2列目(数量)
            amount = row.split(']')[1].trim().replace('[', '').replace('個', '')
            // 3列目(商品名)
            name = row.split(']')[2].trim()

            return {
              id,
              name,
              amount
            }
          })

        // 集計
        const total = inputTable.reduce((result, current) => {
          const element = result.find((pre) => pre.id === current.id)
          if (element) {
            element.amount += Number(current.amount)
          } else {
            result.push({
              id: current.id,
              name: current.name,
              amount: Number(current.amount)
            })
          }
          return result
        }, [])

        // ソートする
        const sortedTotal = total.sort((a, b) => {
          if (a.id < b.id) {
            return -1
          } else if (a.id > b.id) {
            return 1
          }
          return 0
        })

        // 出力
        this.outputData = sortedTotal
          .map((row) => {
            return (
              '"' + row.id + '",' + '"' + row.name + '","' + row.amount + '"'
            )
          })
          .join('\n')
      } catch (error) {
        alert('入力形式に誤りがあります')
      }
    },
    copy() {
      this.$copyText(this.outputData).then(
        (event) => {
          alert('コピーしました')
        },
        (event) => {
          alert('コピーに失敗しました')
        }
      )
    }
  }
}
</script>
