<script>
import { reactive } from "vue";
export default {
  props: {
    items: {
      type: Array,
      default: [],
    },
    fields: {
      type: Array,
      default: ["Name", "Age", "Payment"],
    },
    labels: {
      type: Array,
      default: [],
    },
    actionList: {
      type: Array,
      default: [],
    },
    activeAction: {
      type: Boolean,
      default: true,
    },
    borderTableAll: {
      type: Boolean,
      default: false,
    },
    selectAll: {
      type: Array,
      default: [],
    },
    cus: {
      type: Array,
      default: [],
    },
    activeCheck: {
      type: Boolean,
      default: true,
    },
    startRow: {
      type: Number,
      default: 1,
    },
    showActionList: {
      type: Array,
      default: [true, true],
    }
  },
  setup(props, ntx) {},
};
</script>

<template>
  <div>
    <table class="my-table mb-2" :class="[borderTableAll ? 'border-table-all' : '']">
      <thead>
        <tr>
          <th>
            <input
              type="checkbox"
              name=""
              id=""
              :checked="selectAll[0].checked == true"
              v-model="selectAll[0].checked"
              @click="$emit('selectAll', selectAll[0].checked)"
              class="d-flex align-items-center size-16"
              v-if="activeCheck == true"
            />
          </th>
          <th>Stt</th>
          <th>Khách hàng</th>
          <th v-for="(value, index) in fields" :key="index">{{ value }}</th>
          <th>Lưu ý</th>
          <th>Trạng thái</th>
          <th v-if="activeAction == true">Hành động</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in items" :key="index">
          <td class="size-16">
            <input
              type="checkbox"
              :checked="item.checked == true"
              v-model="item.checked"
              @click="$emit('selectOne', item._id, item)"
              class="d-flex align-items-center size-16"
              v-if="activeCheck == true"
            />
          </td>
          <td class="size-16">{{ startRow + index }}</td>
          <td class="size-16">{{ cus }}</td>
          <td v-for="(label, index1) in labels" :key="index1" class="size-16">
            {{ item[label] }}
          </td>
          <td class="size-16">{{ item.note == null ? "không có" : item.note }}</td>
          <td class="size-16">{{ item.Status_App.name }}</td>
          <td class="size-16" v-if="activeAction == true">
            <div class="d-flex align-items-center">
              <button
                type="button"
                class="mx-2 format-btn"
                data-toggle="modal"
                data-target="#modal-edit"
                v-if="showActionList[0]"
              >
                <span
                  id="edit"
                  class="material-symbols-outlined d-flex align-content-center"
                  @click="$emit('edit', item, true)"
                >
                  edit
                </span>
              </button>
              <span
                id="delete"
                class="material-symbols-outlined"
                @click="$emit('delete', item._id, item)"
                v-if="showActionList[1]"
              >
                delete
              </span>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <p v-if="items.length == 0" class="text-center mt-2">Không tồn tại bản ghi.</p>
  </div>
</template>

<style scoped>
.my-table {
  width: 100%;
  border-collapse: collapse;
}

.border-table-all {
  border: 1px solid var(--gray);
}

.my-table th,
.my-table td {
  border: 1px solid var(--gray);
  border-left: 0;
  border-right: 0;
  padding: 8px;
}

.my-table th {
  font-weight: 900;
  font-size: 14px;
}

.my-table tbody tr:nth-child(even) {
  font-size: 13px;
}

#view,
#edit,
#delete,
#appointment {
  font-size: 18px;
  cursor: pointer;
  border: 1px solid var(--gray);
  border-radius: 4px;
  padding: 1px;
}
#view:hover {
  color: var(--blue);
}
#edit:hover {
  color: var(--yellow);
}
#delete:hover {
  color: var(--red);
}

.takingCare {
  color: green;
}
.takeCare {
  color: red;
}
</style>
