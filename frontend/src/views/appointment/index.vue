<script>
import { useRouter, useRoute } from "vue-router";
import Table from "../../components/table/table-appointment.vue";
import Pagination from "../../components/table/pagination_duy.vue";
import Dropdown from "../../components/form/dropdown.vue";
import Select from "../../components/form/select.vue";
import Search from "../../components/form/search.vue";
import SelectFilter from "../../components/form/select_task_truc.vue";
import InputFilter from "../../components/form/form_filter_truc.vue";
import Add from "../appointment/add.vue";
import Edit from "./edit.vue";
import View from "./view.vue";
import Select_Advanced from "../../components/form/select_advanced.vue";
import Swal from "sweetalert2";
import { reactive, computed, watch, ref, onBeforeMount, onMounted } from "vue";
// services

import Task from "../../services/task.service";
import Cycle from "../../services/cycle.service";
import Employee from "../../services/employee.service";
import Customer from "../../services/customer.service";
import Employees_Task from "../../services/task_employee.service";
import Appointment from "../../services/appointment.service";
import Status_App from "../../services/status_app.service";
import StatusTask from "../../services/status_task.service";
import Evaluate from "../../services/evaluate.service";
import AddAppointment from "../appointment/add.vue";
import {
  http_getAll,
  http_create,
  http_getOne,
  http_deleteOne,
  http_update,
} from "../../assets/js/common.http";
import {
  alert_success,
  alert_error,
  alert_delete,
  alert_warning,
  alert_delete_wide,
} from "../../assets/js/common.alert";

import {
  isDeleteAppointment,
  isCreateAppointment,
  isEditAppointment,
  isReadAppointment,
} from "../../use/getSessionItem";

import { formatDate, formatDateTime } from "../../assets/js/common";
export default {
  components: {
    Table,
    Pagination,
    Dropdown,
    Select,
    Search,
    SelectFilter,
    InputFilter,
    Select_Advanced,
    Add,
    Edit,
    View,
  },
  setup(ctx) {
    const rs = isReadAppointment()
    
    const data = reactive({
      items: [
        {
          _id: "",
          date_time: "",
          content: "",
          note: "",
          place: "",
          StatusAppId: "",
          Status_App: {
            _id: "",
            name: "",
          },
        },
      ],
      entryValue: 5,
      numberOfPages: 1,
      totalRow: 0,
      startRow: 0,
      endRow: 0,
      currentPage: 1,
      searchText: "",
      activeEdit: false,
      selectAll: [
        {
          checked: false,
        },
      ],
      costomerName: "",
      editValue: {
        _id: "",
        date_time: "",
        place: "",
        content: "",
        note: "",
        StatusAppId: "",
        Status_App: {
          _id: "",
          name: "",
        },
      },
      customer: {
        name: "",
      },
      task: [
        {
          _id: "",
          start_date: "",
          end_date: "",
          content: "",
          Customer: {
            _id: "",
            name: "",
          },
          cycleId: "",
          Cycle: {
            _id: "",
            name: "",
          },
          Employee: {
            _id: "",
            name: "",
          },
          Status_Task: {
            _id: "",
            name: "",
          },
          Appointments: {
            _id: "",
            date_time: "",
            content: "",
            Status_App: {
              _id: "",
              name: "",
            },
          },
          Evaluate: {
            _id: "",
            star: "",
          },
          Comment: {
            _id: "",
            content: "",
          },
        },
      ],
      employee: {},
      resetDataEdit: false,
    });
    const route = useRoute();
    const router = useRouter();
    const params = route.params.id;
    const arrayCheck = reactive({ data: [] });
    // computed
    const toString = computed(() => {
     
      if (data.choseSearch == "status") {
        return data.items.map((value, index) => {
          return [value.Status_App.name].join("").toLocaleLowerCase();
        });
      } else if (data.choseSearch == "date_time") {
        return data.items.map((value, index) => {
          return [value.date_time].join("").toLocaleLowerCase();
        });
      }
    });
    const filter = computed(() => {
      return data.items.filter((value, index) => {
        return toString.value[index].includes(
          data.searchText.toLocaleLowerCase()
        );
      });
    });
    const filtered = computed(() => {
      if (!data.searchText) {
        data.totalRow = data.items.length;
        return data.items;
      } else {
        data.totalRow = filter.value.length;
        return filter.value;
      }
    });
    const setNumberOfPages = computed(() => {
      return Math.ceil(filtered.value.length / data.entryValue);
    });
    const setPages = computed(() => {
      if (data.items.length > 0) {
        if (setNumberOfPages.value == 0 || data.entryValue == "All") {
          data.entryValue = data.items.length;
          data.numberOfPages = 1;
        } else data.numberOfPages = setNumberOfPages.value;
        data.startRow = (data.currentPage - 1) * data.entryValue + 1;
        data.endRow = data.currentPage * data.entryValue;
        return filtered.value.filter((item, index) => {
          return (
            index + 1 > (data.currentPage - 1) * data.entryValue &&
            index + 1 <= data.currentPage * data.entryValue
          );
        });
      } else return data.items.value;
    });

    const status_apps = reactive({ status_app: [] });
    const entryValueStatus = ref(""); //id
    const entryNameStatus = ref("Trạng thái");
    const updateEntryValueStatus = (value) => {
      entryValueStatus.value = value;
    };

    watch(entryValueStatus, async (newValue, oldValue) => {
      data.items = await Appointment.findAllAppointment(params);
      for (const value of data.items) {
        value.date_time_format = formatDateTime(value.date_time);
      }
      if (newValue == "") {
        await refresh();
        return;
      } else {
        data.items = data.items.filter((value, index) => {
          
          return value.Status_App._id == entryValueStatus.value;
        });
      }
      
      for (let value of data.items) {
        if (value.note == null || value.note.length == 0) {
          value.note = "không có";
        } else value.note = value.note;
      }

      data.selectAll[0].checked = false;
      for (let value of data.items) {
        value.checked = false;
      }
      for (let value of data.items) {
        for (let array of arrayCheck.data) {
          
          if (array._id == value._id) {
            value.checked = true;
            break;
          }
          value.checked = false;
        }
      }
      data.currentPage = 1;
    });

    // // methods
    const create = async () => {
      
      await refresh();
    };

    const update = async (item) => {
      const result = await http_update(Appointment, editValue._id, editValue);
      if (!result.error) {
        alert_success(`Sửa phân công`, `${result.msg}`);
        refresh();
      } else if (result.error) {
        alert_error(`Sửa phân công`, `${result.msg}`);
      }
    };

    const edit = async (editValue) => {
      
      data.resetDataEdit = false;
   
      editValue.loginId = sessionStorage.getItem("employeeId");
      const result = await http_update(Appointment, editValue._id, editValue);
    
      if (!result.error) {
        alert_success(`Sửa lịch hẹn`, `${result.msg}`);
        refresh();
      } else if (result.error) {
        alert_error(`Sửa lịch hẹn`, `${result.msg}`);
      }
    };

    const refresh = async () => {
      // data.evaluate = await http_getAll(Evaluate);
      data.customer = await http_getOne(Task, params);
      data.task = await http_getOne(Task, params);
      data.customer = data.customer.Customer.name;
      data.items = await Appointment.findAllAppointment(params);
      
      status_apps.status_app = await http_getAll(Status_App);

      for (let value of data.items) {
        if (value.note == null || value.note.length == 0) {
          value.note = "không có";
        } else value.note = value.note;
      }
      
      for (const value of data.items) {
        value.date_time_format = formatDateTime(value.date_time);
      }
      data.items = data.items.map((value, index) => {
        return {
          ...value,
          value: value._id,
        };
      });
      status_apps.status_app = status_apps.status_app.map((value, index) => {
        return {
          ...value,
          value: value._id,
        };
      });

      for (let value of data.items) {
        value.checked = false;
      }
      for (let value of data.items) {
        for (let array of arrayCheck.data) {
          
          if (array._id == value._id) {
            value.checked = true;
            break;
          }
          value.checked = false;
        }
      }

      data.selectAll[0].checked = false;
    };
    const handleSelectAll = (value) => {
      arrayCheck.data = [];
      if (value == false) {
        for (let value1 of data.items) {
          value1.checked = true;
          arrayCheck.data.push(value1);
        }
      } else {
        for (let value1 of data.items) {
          value1.checked = false;
          const index = arrayCheck.data.indexOf(value1._id);
          if (index !== -1) {
            arrayCheck.data.splice(index, 1);
          }
        }
      }
     
    };
    const handleSelectOne = (id, item) => {
      if (item.checked == false) {
        arrayCheck.data.push(item);
      } else {
        arrayCheck.data = arrayCheck.data.filter((value, index) => {
      
          return value._id != id;
        });
      }
      
      data.selectAll[0].checked = false;
    };

    const handleDelete = async (id, item) => {
      
      const isConfirmed = await alert_delete(
        "Xóa",
        `Bạn có chắc là xóa lịch hẹn ${item.date_time_format} không!!`
      );
      if (isConfirmed) {
        const loginId = reactive({ loginId: "", id: "" });
        loginId.loginId = sessionStorage.getItem("employeeId");
        loginId.id = id;
        // 1***** xem thay đổi Appoiment cho phù hợp
        const rsAppointment = await Appointment.deleteOne(id, loginId);
        
        if (rsAppointment.error) {
          alert_error("Lỗi ", rsAppointment.msg);
        } else {
          await refresh();
          alert_success(
            "Thành công",
            `Xóa lịch hẹn ngày ${formatDateTime(
              rsAppointment.document.date_time.toUpperCase()
            )} với khách hàng  ${data.customer}`

          );
        }
      }
    };
    //XÓA NHIỀU
    const deleteMany = async () => {
      
      try {
        //Mảng lịch hẹn sẽ xóa
        if (arrayCheck.data.length == 0) {
          alert_warning("Bạn chưa chọn lịch hẹn", "");
          return;
        }
        let contentAlert = `<p>Bạn có muốn xoá tất cả lịch hẹn này không?
          </p><p>Tổng số lịch hẹn sẽ xoá là:
           <span style="color: blue;">${arrayCheck.data.length}</span></p>
            <table class="table table-bordered">
        <thead>
          <tr>
            <th>Ngày hẹn</th>
            <th>Nội dung</th>
            <th>Địa điểm</th>
            <th>Chú thích</th>
            <th>Trạng thái</th>
          </tr>
        </thead> <tbody>`;
        for (let value of arrayCheck.data) {
          for (const value of data.items) {
            value.date_time_format = formatDateTime(value.date_time);
          }
          
          contentAlert += `<tr>
            <td>${value.date_time_format}</td>
            <td>${value.content}</td>
            <td>  ${value.place} </td>
            <td>  ${value.note}</td>
            <td>  ${value.Status_App.name}</td>
          </tr>`;
        }
        contentAlert += `</tbody>
      </table>`;
        const isConfirmed = await alert_delete_wide(
          `Xoá nhiều lịch hẹn`,
          contentAlert
        );
        if (isConfirmed) {
          let checkDeleteAll = false;
          const loginId = reactive({ loginId: "", id: "" });
          loginId.loginId = sessionStorage.getItem("employeeId");
          for (let valueDelete of arrayCheck.data) {
            // 1***** xem thay đổi Appoiment cho phù hợp
            loginId.id = valueDelete._id;
            const rsAppointment = await Appointment.deleteOne(
              valueDelete._id,
              loginId
            );
            if (rsAppointment.error) {
              alert_error("Lỗi ", rsAppointment.msg);
              checkDeleteAll = false;
            } else {
              checkDeleteAll = true;
            }
          }
          if (checkDeleteAll) {
            await refresh();
            alert_success("Thành công", "Xóa lịch hẹn thành công");
          }
        }
      } catch (error) {
        console.log(error);
      }
    };
    // Hàm callback được gọi trước khi component được mount (load)
    onMounted(async () => {
      await refresh();
    });

    return {
      params,
      create,
      update,
      edit,
      setPages,
      data,
      handleSelectOne,
      handleSelectAll,
      deleteMany,
      handleDelete,
      entryValueStatus,
      entryNameStatus,
      updateEntryValueStatus,
      status_apps,
      // phân quyền
      isDeleteAppointment,
      isCreateAppointment,
      isEditAppointment,
      isReadAppointment,
    };
  },
};
</script>
<template>
  <div class="border-box d-flex flex-column ml-2">
    <!-- Filter -->
    <!-- Search -->
    <!-- <div class="border-hr mb-3"></div> -->
    <div class="d-flex menu my-3 mx-3 justify-content-end">
      <router-link :to="{ name: 'Assignment' }">
        <span class="size-17">Phân công</span>
      </router-link>
      <router-link :to="{ name: '' }" class="active-menu">
        <span class="size-17">Lịch hẹn</span>
      </router-link>
    </div>
    <div class="border-hr mb-3"></div>

    <div class="d-flex flex-column mt-3">
      <div class="d-flex">
        <div class="form-group w-100 ml-3">
          <Select
            :title="`Trạng thái`"
            :entryValue="entryNameStatus"
            :options="status_apps.status_app"
            @update:entryValue="
              (value, value1) => (
                updateEntryValueStatus(value), (entryNameStatus = value1.name)
              )
            "
            @refresh="
              (entryNameStatus = 'Trạng thái'), updateEntryValueStatus('')
            "
            style="height: 35px; width: 200px"
          />
        </div>
        <div class="form-group"></div>
      </div>
    </div>
    <div class="border-hr mb-3"></div>
    <div class="d-flex justify-content-between mx-3 mb-3">
      <div class="d-flex justify-content-start">
        <Select
          class="d-flex justify-content-start"
          :options="[
            {
              name: 5,
              value: 5,
            },
            {
              name: 10,
              value: 10,
            },
            {
              name: 20,
              value: 20,
            },
            {
              name: 30,
              value: 30,
            },
          ]"
          style="width: 125px"
          :title="`Số bản ghi`"
          @update:entryValue="(value) => (data.entryValue = value)"
          :entryValue="data.entryValue"
          @refresh="(data.entryValue = 'All'), (data.currentPage = 1)"
        />
        <Search
          class="ml-3"
          style="width: 300px"
          @update:searchText="(value) => (data.searchText = value)"
          :entryValue="data.searchText"
          @choseSearch="
            async (value) => (
              
              (data.choseSearch = value),
              (data.currentPage = 1)
            )
          "
          @refresh="(data.entryValue = 'All'), (data.currentPage = 1)"
          :options="[
            {
              _id: 'date_time',
              name: 'Tìm kiếm theo ngày hẹn',
            },
            {
              _id: 'status',
              name: 'Tìm kiếm theo trạng thái',
            },
          ]"
        />
      </div>
      <div class="d-flex align-items-start">
        <button
          type="button"
          class="btn btn-danger mr-3"
          data-toggle="modal"
          data-target="#model-delete-all"
          @click="deleteMany()"
          :disabled="isDeleteAppointment() ? false : true"
        >
          <span id="delete-all" class="mx-2">Xoá</span>
        </button>
        <!-- <DeleteAll :items="data.items" /> -->
        <button
          type="button"
          class="btn btn-primary"
          data-toggle="modal"
          data-target="#modal-add"
          :disabled="isCreateAppointment() ? false : true"
        >
          <span id="add" class="mx-2">Thêm</span>
        </button>
        <Add
          @create="create"
          :taskId="params"
          :task="data.task"
          :customer="data.customer"
        />
      </div>
    </div>
    <!-- Table -->
    <Table
      :items="setPages"
      :cus="data.customer"
      :fields="['Ngày hẹn', 'Địa điểm', 'Nội dung lịch hẹn']"
      :labels="['date_time_format', 'place', 'content']"
      :selectAll="data.selectAll"
      @selectAll="(value) => handleSelectAll(value)"
      @selectOne="(id, item) => handleSelectOne(id, item)"
      @delete="handleDelete"
      @edit="
        (value, value1) => (
          (data.editValue = value),
          (data.activeEdit = value1),
          (data.editValue.date_time = data.editValue.date_time.toUpperCase())
        )
      "
      :showActionList="[
        isEditAppointment() ? true : false,
        isDeleteAppointment() ? true : false,
      ]"
    />
    <!-- Pagination -->
    <Pagination
      :numberOfPages="data.numberOfPages"
      :totalRow="data.totalRow"
      :startRow="data.startRow"
      :endRow="data.endRow"
      :currentPage="data.currentPage"
      @update:currentPage="(value) => (data.currentPage = value)"
      class="mx-3"
    />
    <Edit
      :item="data.editValue"
      :class="[data.activeEdit ? 'show-modal' : 'd-none']"
      @cancel="data.activeEdit = false"
      @edit="edit(data.editValue)"
    />
  </div>
</template>
<style scoped>
.border-box {
  border: 1px solid var(--gray);
  border-radius: 5px;
}
.menu {
  /* border: 1px solid var(--gray); */
  border-collapse: collapse;
}
.menu a {
  border: 1px solid var(--gray);
  border-collapse: collapse;
  padding: 8px 12px;
  font-size: 15px;
}
.active-menu {
  color: blue;
}
.none-active-menu {
  color: var(--dark);
}
.border-hr {
  border-top: 1px solid var(--gray);
}
#add,
#delete-all {
  font-size: 14px;
}
</style>
