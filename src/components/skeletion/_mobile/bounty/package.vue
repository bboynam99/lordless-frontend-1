<template>
  <div class="skeletion-bounty-package-box">
    <div class="bounty-package-container skeletion-breath">
      <div class="bounty-package-header">
        <p></p>
        <p></p>
      </div>
      <div class="bounty-package-options">
        <p class="package-item-title"></p>
        <div class="d-flex f-align-center package-options-btns">
          <span class="v-flex" v-for="item in 3" :key="item"></span>
        </div>
      </div>
      <ul class="bounty-package-bounties">
        <li class="package-item-title"></li>
        <li v-for="item in 3" :key="item" class="d-flex f-justify-center bounty-package-item">
          <p></p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'bounty-package-skeletion'
}
</script>

<style lang="scss" scoped>
  .skeletion-bounty-package-box {
    margin-top: 44px;
  }
  .bounty-package-container {
    padding: 24px;
  }
  // .bounty-package-header {
  //   >h2 {
  //     margin-bottom: 16px;
  //     width: 50%;
  //     height: 28px;
  //     background-color: $--skeletion-dark;
  //   }
  //   >p {
  //     margin-top: 6px;
  //     height: 16px;
  //     background-color: $--skeletion-dark;
  //     &:last-of-type {
  //       width: 60%;
  //     }
  //   }
  // }
  .bounty-package-header {
    margin: -24px -24px 0;
    padding: 24px 20px;
    background-color: $--skeletion-light;
    >p {
      width: 60%;
      height: 20px;
      background-color: $--skeletion-dark;
      &:nth-of-type(2) {
        margin-top: 10px;
        width: 80%;
      }
    }
  }
  .bounty-package-options {
    margin-top: 24px;
  }
  .package-options-btns {
    margin-top: 18px;
    >span {
      height: 40px;
      background-color: $--skeletion-dark;
      border-radius: 5px;
      &:not(:first-of-type) {
        margin-left: 8px;
      }
    }
  }
  .bounty-package-bounties {
    margin-top: 24px;
  }
  .package-item-title {
    width: 40%;
    height: 24px;
    background-color: $--skeletion-dark;
  }
  .bounty-package-item {
    margin-top: 12px;
    padding: 18px;
    border-radius: 5px;
    background-color: $--skeletion-light;
    >p {
      width: 100%;
      height: 16px;
      background-color: $--skeletion-dark;
    }
  }
</style>
