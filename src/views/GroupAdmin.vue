<template>
  <div type="flex">
    <Row class="row" v-if="groups.length">
      <i-col :key="group.id" :lg="{span:8}" :md="{span:12}" :sm="{span:12}" :xs="{span:24}" style="padding:10px" v-for="group in groups">
        <Card :dis-hover="true">
          <div class="item">
            <span>组名：</span>
            <span>{{group.name}}</span>
          </div>
          <div class="item">
            <span>组长：</span>
            <span class="group-leader-select">
              <i-select v-model="group.leader">
                <i-option :key="user.id" :value="user.id" v-for="user in group.member">{{user.username}}</i-option>
              </i-select>
            </span>
            <i-button :disabled="!group.leader" type="text" icon="close" @click="hanldeEmpty(group)"></i-button>
          </div>
          <div class="item">
            <i-button :disabled="originLeader[group.id] === group.leader" :loading="inSavingMap[group.id]" @click="save(group)" type="primary">保存</i-button>
          </div>
        </Card>
      </i-col>
    </Row>
  </div>
</template>
<script>
import { Row, Col } from 'iview/src/components/grid/';
import Icon from 'iview/src/components/icon';
import Card from 'iview/src/components/card/';
import { Select, Option } from 'iview/src/components/select';
import Form from 'iview/src/components/form/';
import Input from 'iview/src/components/input/input';
import Button from 'iview/src/components/button/button';
import AV from 'leancloud-storage';
import api from '@/api/';
import Message from 'iview/src/components/message';
export default {
  components: {
    Row,
    Card,
    Icon,
    'i-col': Col,
    'i-button': Button,
    'i-select': Select,
    'i-option': Option,
    'i-form': Form,
    FormItem: Form.Item
  },
  data() {
    return {
      groups: [],
      inSavingMap: {},
      originLeader: {}
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    getData() {
      api.getAllUserAsTree().then(r => {
        const res = JSON.parse(JSON.stringify(r));
        const originLeader = {};
        res.forEach(g => {
          if (!g.leader) {
            g.leader = '';
          } else {
            g.leader = g.leader.objectId;
          }
          originLeader[g.id] = g.leader;
          this.inSavingMap[g.id] = false;
        });

        this.$set(this, 'originLeader', originLeader);
        this.$set(this, 'groups', res);
      });
    },
    hanldeEmpty(group) {
      group.leader = '';
    },
    save(group) {
      if (group.leader === this.originLeader[group.id]) {
        return;
      }
      this.inSavingMap[group.id] = true;
      const groupObj = AV.Object.createWithoutData('Group', group.id);
      if(group.leader) {
        const leaderObj = AV.Object.createWithoutData('_User', group.leader);
        groupObj.set('leader', leaderObj);
      }else {
        groupObj.set('leader', null);
      }
      return groupObj.save().then(g => {
        this.originLeader[group.id] = group.leader;
        this.inSavingMap[group.id] = false;
        Message.success({
          content: '保存成功'
        });
      });
    }
  }
};
</script>

<style scoped>
.item {
  margin: 6px 0;
}
.group-leader-select {
  display: inline-block;
  vertical-align: middle;
  width: 200px;
}
</style>
