<template>
  <el-container direction="vertical" >
    <el-row >
      <el-col :span="12">
        <h3 class="title">MOF/COF膜材料性质数据库</h3>
      </el-col>
      <el-col :span="12">
        <el-autocomplete 
          style="width: 100%;padding-top: 20px;"
          v-model="searchKeyword"
          :fetch-suggestions="querySearch"
          :trigger-on-focus="false"
          placeholder="Please input"
          clearable
          @select="handleSelect" 
        >
          <template #default="{ item }">
            <div>{{ item.name }}</div>
          </template>
          <template #append>
              <el-button :icon="Search" @click="molDisplay"/>
          </template>
        </el-autocomplete>
      </el-col>
    </el-row>

    <el-row gutter=2 style="margin-top: 15px;">
      <el-col :span="11">
        <el-menu class="customElMenu" mode="horizontal" default-active="通用数据库" @select="onSelectMenu">
          <el-menu-item index="Structure">通用数据库</el-menu-item>
          <el-sub-menu index="zhuan">
            <template #title>专用数据库</template>
              <el-sub-menu index="Infinite">
                <template #title>无限稀释组分</template>
                  <el-menu-item index="Infinite-ch4">CH₄/N₂</el-menu-item>
                  <el-menu-item index="Infinite-c2h6">C₂H₆/C₂H₄</el-menu-item>
                  <el-menu-item index="Infinite-c3h8">C₃H₈/C₃H₆</el-menu-item>
              </el-sub-menu>
              <el-sub-menu index="Mixing">
                <template #title>混合组分</template>
                <el-menu-item index="Mixing-ch4">CH₄/N₂</el-menu-item>
                <el-menu-item index="Mixing-c2h6">C₂H₆/C₂H₄</el-menu-item>
                <el-menu-item index="Mixing-c3h8">C₃H₈/C₃H₆</el-menu-item>
              </el-sub-menu>
          </el-sub-menu>
        </el-menu>

        <!-- Structure -->
        <el-table :data="table1" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu === 'Structure'">
          <el-table-column prop="name1" label="性质" width="120%" />
          <el-table-column prop="name2" label="属性" width="170%"/>
          <el-table-column prop="data" label="数值" />
          <el-table-column prop="danwei" label="单位" />
        </el-table>

        <!-- Infinite -->
        <div v-if="selectedMenu.includes('Infinite')">
          <div v-if="hasData_rd == 'true'">
            <el-table :data="table2_1" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu.includes('ch4')">
              <el-table-column prop="name1" label="性质" width="120%" />
              <el-table-column prop="name2" label="属性" width="170%"/>
              <el-table-column prop="data" label="数值" />
              <el-table-column prop="danwei" label="单位" />
            </el-table>
            <el-table :data="table2_2" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu.includes('c2h6')">
              <el-table-column prop="name1" label="性质" width="120%" />
              <el-table-column prop="name2" label="属性" width="170%"/>
              <el-table-column prop="data" label="数值" />
              <el-table-column prop="danwei" label="单位" />
            </el-table>
            <el-table :data="table2_3" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu.includes('c3h8')">
              <el-table-column prop="name1" label="性质" width="120%" />
              <el-table-column prop="name2" label="属性" width="170%"/>
              <el-table-column prop="data" label="数值" />
              <el-table-column prop="danwei" label="单位" />
            </el-table>
          </div>
          <div v-else>
            <div style="margin-top: 20px;">
              No Data
            </div>
          </div>
        </div>

        <!-- Mixing -->
        <div v-if="selectedMenu.includes('Mixing')">
          <div v-if="hasData_hun == 'true'">
            <el-table :data="table3_1" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu.includes('ch4')">
              <el-table-column prop="name1" label="性质" width="120%" />
              <el-table-column prop="name2" label="属性" width="170%"/>
              <el-table-column prop="data" label="数值" />
              <el-table-column prop="danwei" label="单位" />
            </el-table>
            <el-table :data="table3_2" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu.includes('c2h6')">
              <el-table-column prop="name1" label="性质" width="120%" />
              <el-table-column prop="name2" label="属性" width="170%"/>
              <el-table-column prop="data" label="数值" />
              <el-table-column prop="danwei" label="单位" />
            </el-table>
            <el-table :data="table3_3" border style="width: 100%" size="small" :table-layout="fixed" v-if="selectedMenu.includes('c3h8')">
              <el-table-column prop="name1" label="性质" width="120%" />
              <el-table-column prop="name2" label="属性" width="170%"/>
              <el-table-column prop="data" label="数值" />
              <el-table-column prop="danwei" label="单位" />
            </el-table>
          </div>
          <div v-else>
            <div style="margin-top: 20px;">
              No Data
            </div>
          </div>
        </div>

      </el-col>

      <el-col :span="1">
      </el-col>

      <el-col :span="12">
          <el-row style="margin-top: 5px;">
            <!-- 选择背景颜色 -->
            <el-col :span="5">
              <el-select v-model="bgColor" placeholder="BG Color" class="btn">
              <el-option
                v-for="item in bgColors"
                :key="item.value"
                :label="item.label"
                :value="item.value"
                @click="bgColorDisplay"
              />
              </el-select>
            </el-col>
            <!-- 选择分子样式 style,options,styleDisplay 4,2 -->
            <el-col :span="5">
              <el-select v-model="style" placeholder="Style" class="btn">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
                @click="styleDisplay"
              />
              </el-select>
            </el-col>
            <!-- lable -->
            <el-col :span="5">
              <el-select v-model="selectedElem" placeholder="Lable" class="btn">
              <el-option
                v-for="item in Elements"
                :key="item.value"
                :label="item.label"
                :value="item.value"
                @click="LableDisplay"
              />
              </el-select>
            </el-col>
            <!-- 居中 -->
            <el-col :span="5">
              <el-button class="btn" @click="MolRecenter">Recenter</el-button>
            </el-col>
            <!-- 下载 -->
            <el-col :span="4">
              <el-button class="btn" @click="MolDownload">Download</el-button>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <div id="gldiv" class="mol-container"></div>
            </el-col>
          </el-row>
      </el-col>
    </el-row>
  
  </el-container>
</template>
  




  
<script setup>
  import { onMounted, ref } from 'vue'
  import { Search } from '@element-plus/icons-vue'
  import axios from 'axios';

  // 变量
  const searchKeyword = ref('')
  var searchMol = ref('')
  var allFileNames = ref([])
  var viewer = ref('')
  
  const name=ref('')
  const file_path=ref('')
  const file_type=ref('')
  var content = ref('')

  //初始值 无编号 旧名 文件路径 文件类型 
  var hasData_rd=ref('')
  var hasData_hun=ref('')
  var property = ref({})
  var property_rd = ref({})
  var property_hun = ref({})

  const table1 = ref([]);
  const table2_1 = ref([]);
  const table2_2 = ref([]);
  const table2_3 = ref([]);
  const table3_1 = ref([]);
  const table3_2 = ref([]);
  const table3_3 = ref([]);




  const selectedElem = ref('')
  const Elements =[
    {value:'false',label:'No lables'},
    {value:'H',label:'H'},
    {value:'Li',label:'Li'},
    {value:'Be',label:'Be'},
    {value:'B',label:'B'},
    {value:'C',label:'C'},
    {value:'N',label:'N'},
    {value:'O',label:'O'},
    {value:'F',label:'F'},
    {value:'Na',label:'Na'},
    {value:'Mg',label:'Mg'},
    {value:'Al',label:'Al'},
    {value:'Si',label:'Si'},
    {value:'P',label:'P'},
    {value:'S',label:'S'},
    {value:'Cl',label:'Cl'},
    {value:'K',label:'K'},
    {value:'Ca',label:'Ca'},
    {value:'Sc',label:'Sc'},
    {value:'Ti',label:'Ti'},
    {value:'V',label:'V'},
    {value:'Cr',label:'Cr'},
    {value:'Mn',label:'Mn'},
    {value:'Fe',label:'Fe'},
    {value:'Co',label:'Co'},
    {value:'Ni',label:'Ni'},
    {value:'Cu',label:'Cu'},
    {value:'Zn',label:'Zn'},
    {value:'Ga',label:'Ga'},
    {value:'Ge',label:'Ge'},
    {value:'As',label:'As'},
    {value:'Se',label:'Se'},
    {value:'Br',label:'Br'},
    {value:'In',label:'In'},
    {value:'Fr',label:'Fr'},
    {value:'U',label:'U'},
  ]

  const bgColor = ref('')
  const bgColors = [
    {value: 'white',label: 'white',},
    {value: 'lightgrey',label: 'lightgrey',},
    {value: 'grey',label: 'grey',},
    {value: 'black',label: 'black',},
    {value: 'blue',label: 'blue',},
  ]

  const style = ref('')//选择器值
  const options = [
    {value: 'line',label: 'line',},
    {value: 'stick',label: 'stick',},
    {value: 'cross',label: 'cross',},
    // {value: 'cartoon',label: 'cartoon',},
    {value: 'sphere',label: 'sphere',},
  ]
  const selectedMenu = ref('Structure');
  
  

  
  // 生命周期钩子
  onMounted(() => {
    console.log(`the component is now mounted.`)

    //初始化数据
    initMolDisplay()

    //支持模糊搜索
    getAllFileNames()
  })



  // 模糊查询
  async function getAllFileNames(){
    // 存在response中
    let response = ref('')
    try {
      response = await axios.get('/apiTest/getAllNames');
      console.log(response.data)
    } catch (error) {
        console.error(error);
    }
    allFileNames.value = response.data
  }
  function createFilter(file,queryString){
    return file.name.toLowerCase().indexOf(queryString.toLowerCase()) === 0
  }
  //根据输入的queryString，空则返回所有restaurants，否则返回所有匹配的restaurants
  const querySearch = (queryString,cb) => {
    const results = queryString
      ? allFileNames.value.filter((file)=>createFilter(file,queryString))
      : allFileNames.value
    // call callback function to return suggestions
    cb(results)
  }
  const handleSelect = (item) => {
    searchKeyword.value = item.name
    searchMol.value = item.name
    // molDisplay()
  }


  

  // 展示分子结构
  async function initMolDisplay(){
    searchMol.value="M3027"//必须是cif文件
    await getMolContent() 
    //ThreeDMol
    //先创建
    viewer = $3Dmol.createViewer(
     'gldiv', //id of div to create canvas in
     {
       defaultcolors: $3Dmol.elementColors.rasmol,
       backgroundColor: 'white'
     }
    );
    //再展示
    viewer.addModel(content.value, "cif");
    // receptorModel = m = glviewer.addModel(content.value, "cif");
    // atoms = m.selectedAtoms({});
    // for ( var i in atoms) {
    //   var atom = atoms[i];
    //   atom.clickable = true;
    //   atom.callback = atomcallback;
    // }
    viewer.mapAtomProperties($3Dmol.applyPartialCharges);
    viewer.setStyle({}, {stick: {}});    
    viewer.addUnitCell(viewer, {box:{color:'red'},alabel:'X',blabel:'Y',clabel:'Z'});
    viewer.zoomTo();
    viewer.render();
  }
  async function molDisplay(){
    await getMolContent()        // 根据path获取分子文件内容content
    cifDisplay()
    // threeDMol()               // 调用3dmol.js库展示分子
  }
  async function getMolContent(){
    await getMysqlMol()   
    await getMysqlRd()
    await getMysqlHun()
    await axios.get(file_path.value).then((response) => {
      content.value = response.data
    })
  }
  async function getMysqlMol(){
    let response = ref('')
    //axios
    try {
      response = await axios.get(
        '/apiTest/getByName/'+searchMol.value);
          property.value = response.data[0];
          name.value = property.value.name;
          file_path.value = property.value.file_path;
          file_type.value = property.value.file_type;
          //保留三位小数
          floatFormat(property,3)

          table1.value = [
          {
            name1: 'Name',
            name2: '名称',
            data: property.value.name,
            danwei: '',
          },
          {
            name1: 'Dimension',
            name2: '结构维数',
            data: property.value.type,
            danwei: '',
          },
          {
            name1: 'Topology',
            name2: '拓扑类型',
            data: property.value.topology,
            danwei: '',
          },
          {
            name1: 'Volume',
            name2: '晶胞体积',
            data: property.value.vol,
            danwei: 'cm³/g',
          },
          {
            name1: 'Density',
            name2: '晶体密度',
            data: property.value.den,
            danwei: 'g/cm³',
          },
          {
            name1: 'LCD',
            name2: '最大孔腔直径',
            data: property.value.LCD,
            danwei: 'Å',
          },
          {
            name1: 'PLD',
            name2: '最大限制直径',
            data: property.value.PLD,
            danwei: 'Å',
          },
          {
            name1: 'VSA',
            name2: '单位体积比表面积',
            data: property.value.Sa_m,
            danwei: 'm²/cm³',
          },
          {
            name1: 'GSA',
            name2: '单位重量比表面积',
            data: property.value.Sa_g,
            danwei: 'm²/g',
          },
          {
            name1: 'Pore Volume (Vp)',
            name2: '单位重量孔体积',
            data: property.value.Va,
            danwei: 'cm³/g',
          },
          {
            name1: 'Void Fraction (φ)',
            name2: '孔隙率',
            data: property.value.phi,
            danwei: 'cm³/cm³',
          },
        ]

      
    } catch (error) {
        console.error(error);
    }
  }
  function floatFormat(object,n){
    for (const key in object.value) {
      var number = object.value[key]
      if (typeof(number) === 'number') {
        if (number === 0) {} 
        else if (Math.abs(number) < 0.001) {
          object.value[key] =  number.toExponential(n);
        } else {
          // 否则，保留三位小数
          object.value[key] = number.toFixed(n);
        }
      }
    }
  }
  async function getMysqlRd(){
    let response = ref('')
    //axios
    try {
      response = await axios.get(
        '/apiTest/getByName_rd/'+searchMol.value); 
        if (typeof response.data[0] === 'undefined')
          hasData_rd.value="false"
        else {
          hasData_rd.value="true"
          property_rd.value = response.data[0]
        }
        //保留三位小数
        floatFormat(property_rd,3)     
        
        table2_1.value = [
        {
          name1: 'CH₄_KH',
          name2: 'CH₄的亨利系数',
          data: property_rd.value.CH4_Hen,
          danwei: 'mol/kg/Pa',
        },
        {
          name1: 'CH₄_Qst⁰',
          name2: 'CH₄的吸附热',
          data: property_rd.value.CH4_Inf,
          danwei: 'KJ/mol',
        },
        {
          name1: 'CH₄_Ds',
          name2: 'CH₄的自扩散系数',
          data: property_rd.value.CH4_Ds,
          danwei: 'm²/s',
        },
        {
          name1: 'CH₄_P0',
          name2: 'CH₄的渗透通量',
          data: property_rd.value.CH4_P0,
          danwei: 'barrer',
        },
        {
          name1: 'N₂_KH',
          name2: 'N₂的亨利系数',
          data: property_rd.value.N2_Hen,
          danwei: 'mol/kg/Pa',
        },
        {
          name1: 'N₂_Qst⁰',
          name2: 'N₂的吸附热',
          data: property_rd.value.N2_Inf,
          danwei: 'KJ/mol',
        },
        {
          name1: 'N₂_Ds',
          name2: 'N₂的自扩散系数',
          data: property_rd.value.N2_Ds,
          danwei: 'm²/s',
        },
        {
          name1: 'N₂_P0',
          name2: 'N₂的渗透通量',
          data: property_rd.value.N2_P0,
          danwei: 'barrer',
        },
        {
          name1: 'Sads_CH₄/N₂',
          name2: 'CH₄/N₂的吸附选择性',
          data: property_rd.value.Sads_CH4_N2,
          danwei: ' ',
        },
        {
          name1: 'Sdiff_N₂/CH₄',
          name2: 'N₂/CH₄的扩散选择性',
          data: property_rd.value.Sdiff_N2_CH4,
          danwei: ' ',
        },
        {
          name1: 'Smem_CH₄/N₂',
          name2: 'CH₄/N₂的膜分离选择性',
          data: property_rd.value.Sperm_CH4_N2,
          danwei: ' ',
        },
        // {
        //   name1: 'Smem_N₂/CH₄',
        //   name2: 'N₂/CH₄的膜分离选择性',
        //   data: property_rd.value.Sperm_N2_CH4,
        //   danwei: ' ',
        // },
        ]

        table2_2.value = [
        {
          name1: 'C₂H₆_KH',
          name2: 'C₂H₆的亨利系数',
          data: property_rd.value.C2H6_Hen,
          danwei: 'mol/kg/Pa',
        },
        {
          name1: 'C₂H₆_Qst⁰',
          name2: 'C₂H₆的吸附热',
          data: property_rd.value.C2H6_Inf,
          danwei: 'KJ/mol',
        },
        {
          name1: 'C₂H₆_Ds',
          name2: 'C₂H₆的自扩散系数',
          data: property_rd.value.C2H6_Ds,
          danwei: 'm²/s',
        },
        {
          name1: 'C₂H₆_P0',
          name2: 'C₂H₆的渗透通量',
          data: property_rd.value.C2H6_P0,
          danwei: 'barrer',
        },
        {
          name1: 'C₂H₄_KH',
          name2: 'C₂H₄的亨利系数',
          data: property_rd.value.C2H4_Hen,
          danwei: 'mol/kg/Pa',
        },
        {
          name1: 'C₂H₄_Qst⁰',
          name2: 'C₂H₄的吸附热',
          data: property_rd.value.C2H4_Inf,
          danwei: 'KJ/mol',
        },
        {
          name1: 'C₂H₄_Ds',
          name2: 'C₂H₄的自扩散系数',
          data: property_rd.value.C2H4_Ds,
          danwei: 'm²/s',
        },
        {
          name1: 'C₂H₄_P0',
          name2: 'C₂H₄的渗透通量',
          data: property_rd.value.C2H4_P0,
          danwei: 'barrer',
        },
        {
          name1: 'Sads_C₂H₆/C₂H₄',
          name2: 'C₂H₆/C2H4的吸附选择性',
          data: property_rd.value.Sads_C2H6_C2H4,
          danwei: ' ',
        },
        {
          name1: 'Sdiff C₂H₄/C₂H₆',
          name2: 'C₂H₄/C₂H₆的扩散选择性',
          data: property_rd.value.Sdiff_C2H4_C2H6,
          danwei: ' ',
        },
        {
          name1: 'Smem_C₂H₆/C₂H₄',
          name2: 'C₂H₆/C₂H₄的膜分离选择性',
          data: property_rd.value.Sperm_C2H6_C2H4,
          danwei: ' ',
        },
        // {
        //   name1: 'Smem_C₂H₄/C₂H₆',
        //   name2: 'C₂H₄/C₂H₆的膜分离选择性',
        //   data: property_rd.value.Sperm_C2H4_C2H6,
        //   danwei: ' ',
        // },
        ]

        table2_3.value = [
        {
          name1: 'C₃H₈_KH',
          name2: 'C₃H₈的亨利系数',
          data: property_rd.value.C3H8_Hen,
          danwei: 'mol/kg/Pa',
        },
        {
          name1: 'C₃H₈_Qst⁰',
          name2: 'C₃H₈的吸附热',
          data: property_rd.value.C3H8_Inf,
          danwei: 'KJ/mol',
        },
        {
          name1: 'C₃H₈_Ds',
          name2: 'C₃H₈的自扩散系数',
          data: property_rd.value.C3H8_Ds,
          danwei: 'm²/s',
        },
        {
          name1: 'C₃H₈_P0',
          name2: 'C₃H₈的渗透通量',
          data: property_rd.value.C3H8_P0,
          danwei: 'barrer',
        },
        {
          name1: 'C₃H₆_KH',
          name2: 'C₃H₆的亨利系数',
          data: property_rd.value.C3H6_Hen,
          danwei: 'mol/kg/Pa',
        },
        {
          name1: 'C₃H₆_Qst⁰',
          name2: 'C₃H₆的吸附热',
          data: property_rd.value.C3H6_Inf,
          danwei: 'KJ/mol',
        },
        {
          name1: 'C₃H₆_Ds',
          name2: 'C₃H₆的自扩散系数',
          data: property_rd.value.C3H6_Ds,
          danwei: 'm²/s',
        },
        {
          name1: 'C₃H₆_P0',
          name2: 'C₃H₆的渗透通量',
          data: property_rd.value.C3H6_P0,
          danwei: 'barrer',
        },
        {
          name1: 'Sads_C₃H₈/C₃H₆',
          name2: 'C₃H₈/C3H6的吸附选择性',
          data: property_rd.value.Sads_C3H8_C3H6,
          danwei: ' ',
        },
        {
          name1: 'Sdiff C₃H₆/C₃H₈',
          name2: 'C₃H₆/C₃H₈的扩散选择性',
          data: property_rd.value.Sdiff_C3H6_C3H8,
          danwei: ' ',
        },
        {
          name1: 'Smem_C₃H₈/C₃H₆',
          name2: 'C₃H₈/C₃H₆的膜分离选择性',
          data: property_rd.value.Sperm_C3H8_C3H6,
          danwei: ' ',
        },
        // {
        //   name1: 'Smem_C₃H₆/C₃H₈',
        //   name2: 'C₃H₆/C₃H₈的膜分离选择性',
        //   data: property_rd.value.Sperm_C3H6_C3H8,
        //   danwei: ' ',
        // },
        ]


    } catch (error) {
        console.error(error);
    }
  }
  async function getMysqlHun(){
    let response = ref('')
    //axios
    try {
      response = await axios.get(
        '/apiTest/getByName_hun/'+searchMol.value);
        if (typeof(response.data[0]) === 'undefined'){
          hasData_hun.value="false"
        }
        else {
          hasData_hun.value="true"
          property_hun.value = response.data[0]
        }
        //保留三位小数
        floatFormat(property_hun,3)
        console.log(property_hun)

        table3_1.value = [
          {
            name1: 'CH₄_UC',
            name2: '单个晶胞CH₄的吸附分子数',
            data: property_hun.value.CH4_UC,
            danwei: 'molecules/u.c.',
          },
          {
            name1: 'CH₄_Qst⁰',
            name2: 'CH₄吸附热',
            data: property_hun.value.CH4_Qst,
            danwei: 'KJ/mol',
          },
          {
            name1: 'CH₄_Ds',
            name2: 'CH₄的自扩散系数',
            data: property_hun.value.CH4_Ds,
            danwei: 'm²/s',
          },
          {
            name1: 'P_CH₄',
            name2: 'CH₄的渗透通量',
            data: property_hun.value.P_CH4,
            danwei: 'barrer',
          },
          {
            name1: 'N₂_UC',
            name2: '单个晶胞N₂的吸附分子数',
            data: property_hun.value.N2_UC,
            danwei: ' molecules/u.c.',
          },
          {
            name1: 'N₂_Qst⁰',
            name2: 'N₂吸附热',
            data: property_hun.value.N2_Qst,
            danwei: ' KJ/mol ',
          },
          {
            name1: 'N₂_Ds',
            name2: 'N₂的自扩散系数',
            data: property_hun.value.N2_Ds,
            danwei: 'm²/s',
          },
          {
            name1: 'P_N₂',
            name2: 'N₂的渗透通量',
            data: property_hun.value.P_N2,
            danwei: 'barrer',
          },
          {
            name1: 'Sads_CH₄/N₂',
            name2: 'CH₄/N₂的吸附选择性',
            data: property_hun.value.Sads_CH4_N2,
            danwei: '',
          },
          {
            name1: 'Sdiff_CH₄/N₂',
            name2: 'CH₄/N₂的扩散选择性',
            data: property_hun.value.Sdiff_CH4_N2,
            danwei: '',
          },
          {
            name1: 'Smem_N₂/CH₄',
            name2: 'CH₄/N₂的膜分离选择性',
            data: property_hun.value.Sperm_N2_CH4,
            danwei: '',
          },
        ]
        table3_2.value = [
          {
            name1: 'C₂H₆_UC',
            name2: '单个晶胞C₂H₆的吸附分子数',
            data: property_hun.value.C2H6_UC,
            danwei: ' molecules/u.c.',
          },
          {
            name1: 'C₂H₆_Qst⁰',
            name2: 'C₂H₆吸附热',
            data: property_hun.value.C2H6_Qst,
            danwei: 'KJ/mol',
          },
          {
            name1: 'C₂H₆_Ds',
            name2: 'C₂H₆的自扩散系数',
            data: property_hun.value.C2H6_Ds,
            danwei: 'm²/s',
          },
          {
            name1: 'P_C₂H₆',
            name2: 'C₂H₆的渗透通量',
            data: property_hun.value.P_C2H6,
            danwei: 'barrer',
          },
          {
            name1: 'C₂H₄_UC',
            name2: '单个晶胞C₂H₄的吸附分子数',
            data: property_hun.value.C2H4_UC,
            danwei: ' molecules/u.c.',
          },
          {
            name1: 'C₂H₄_Qst⁰',
            name2: 'C₂H₄吸附热',
            data: property_hun.value.C2H4_Qst,
            danwei: 'KJ/mol',
          },
          {
            name1: 'C₂H₄_Ds',
            name2: 'C₂H₄的自扩散系数',
            data: property_hun.value.C2H4_Ds,
            danwei: 'm²/s',
          },
          {
            name1: 'P_C₂H₄',
            name2: 'C₂H₄的渗透通量',
            data: property_hun.value.P_C2H4,
            danwei: 'barrer',
          },
          {
            name1: 'S_ads_C₂H₆/C₂H₄',
            name2: 'C₂H₆/C₂H₄的吸附选择性',
            data: property_hun.value.Sads_C2H6_C2H4,
            danwei: '',
          },
          {
            name1: 'S_diff_C₂H₆/C₂H₄',
            name2: 'C₂H₆/C₂H₄的扩散选择性',
            data: property_hun.value.Sdiff_C2H6_C2H4,
            danwei: '',
          },
          {
            name1: 'Smem_C₂H₄/C₂H₆',
            name2: 'C₂H₄/C₂H₆的膜分离选择性',
            data: property_hun.value.Sperm_C2H4_C2H6,
            danwei: '',
          },
        ]

        table3_3.value = [
          {
            name1: 'C₃H₈_UC',
            name2: '单个晶胞C₃H₈的吸附分子数',
            data: property_hun.value.C3H8_UC,
            danwei: ' molecules/u.c.',
          },
          {
            name1: 'C₃H₈_Qst⁰',
            name2: 'C₃H₈吸附热',
            data: property_hun.value.C3H8_Qst,
            danwei: 'KJ/mol',
          },
          {
            name1: 'C₃H₈_Ds',
            name2: 'C₃H₈的自扩散系数',
            data: property_hun.value.C3H8_Ds,
            danwei: 'm²/s',
          },
          {
            name1: 'P_C₃H₈',
            name2: 'C₃H₈的渗透通量',
            data: property_hun.value.P_C3H8,
            danwei: 'barrer',
          },
          {
            name1: 'C₃H₆_UC',
            name2: '单个晶胞C₃H₆的吸附分子数',
            data: property_hun.value.C3H6_UC,
            danwei: ' molecules/u.c.',
          },
          {
            name1: 'C₃H₆_Qst⁰',
            name2: 'C₃H₆吸附热',
            data: property_hun.value.C3H6_Qst,
            danwei: 'KJ/mol',
          },
          {
            name1: 'C₃H₆_Ds',
            name2: 'C₃H₆的自扩散系数',
            data: property_hun.value.C3H6_Ds,
            danwei: 'm²/s',
          },
          {
            name1: 'P_C₃H₆',
            name2: 'C₃H₆的渗透通量',
            data: property_hun.value.P_C3H6,
            danwei: 'barrer',
          },
          {
            name1: 'S_ads_C₃H₈/C₃H₆',
            name2: 'C₃H₈/C₃H₆的吸附选择性',
            data: property_hun.value.Sads_C3H8_C3H6,
            danwei: '',
          },
          {
            name1: 'S_diff_C₃H₈/C₃H₆',
            name2: 'C₃H₈/C₃H₆的扩散选择性',
            data: property_hun.value.Sdiff_C3H8_C3H6,
            danwei: '',
          },
          {
            name1: 'Smem_C₃H₆/C₃H₈',
            name2: 'C₃H₆/C₃H₈的膜分离选择性',
            data: property_hun.value.Sperm_C3H6_C3H8,
            danwei: '',
          },
        ]


    } catch (error) {
        console.error(error);
    }
  }
  function cifDisplay(){
    // 判断type类型，转换3dmol.js不支持的格式
    if(file_type.value == 'car'){
      convertFormat(content.value,'car','xyz')
    }
    else threeDMol()
  }
  function convertFormat(inData,inFormat,outFormat){
    var OpenBabel = OpenBabelModule();
    var outData;
    OpenBabel.onRuntimeInitialized = function()
    {
      // Now the OpenBabel routines can be called 
      var conv = new OpenBabel.ObConversionWrapper();  // create ObConversionWrapper instance
      try
      {
          // var inData = ;   // set input data
          conv.setInFormat('', inFormat);  // set input format by file extension
              // the input format can also be set by MIME type:
              // conv.setInFormat('chemical/x-mdl-molfile');
          var mol = new OpenBabel.OBMol();  // create a new molecule object...
          conv.readString(mol, inData);  // ... and load it with input data
          conv.setOutFormat('', outFormat);  // set out format by file extension
          outData = conv.writeString(mol, false);  // get output data, do not trim white spaces
          content.value = outData
          file_type.value = 'xyz'
          threeDMol()
      }
      finally
      {
        conv.delete();  // free ObConversionWrapper instance
      }
    }
  }
  function threeDMol(){
    //展示参数
    // viewer = $3Dmol.createViewer( "gldiv" );
    viewer.clear();
    viewer.addModel( content.value, file_type.value);/* load data */
    viewer.setStyle({}, {stick: {}});                     /* style all atoms */
    viewer.addUnitCell(viewer, {box:{color:'red'},alabel:'X',blabel:'Y',clabel:'Z'});
    viewer.zoomTo();                                      /* set camera */
    viewer.render();  
  }

  

  


function bgColorDisplay(){
    if(bgColor.value == 'white'){viewer.setBackgroundColor('white')}
    else if(bgColor.value == 'black'){viewer.setBackgroundColor('black')}
    else if(bgColor.value == 'blue'){viewer.setBackgroundColor('blue')}
    else if(bgColor.value == 'grey'){viewer.setBackgroundColor('grey')}
    else if(bgColor.value == 'lightgrey'){viewer.setBackgroundColor('lightgrey')}
    viewer.render();
  }
  function styleDisplay(){
    if(style.value == 'line'){viewer.setStyle({},{line:{}})}
    else if(style.value == 'stick'){viewer.setStyle({},{stick:{}})}
    else if(style.value == 'cross'){viewer.setStyle({},{cross:{}})}
    else if(style.value == 'cartoon'){viewer.setStyle({},{stick:{}})}
    else if(style.value == 'sphere'){viewer.setStyle({},{sphere:{}})}
    viewer.render();
  }
  function LableDisplay(){
    viewer.removeAllLabels();
    if(selectedElem.value != String(false)){
      var atoms = viewer.getModel().selectedAtoms({
        elem:selectedElem.value
      });
      for ( var a in atoms) {
        var atom = atoms[a];

        viewer.addLabel(atom.index, {
          inFront : true,
          showBackground:true,
          fontSize : 10,
          position : {
            x : atom.x,
            y : atom.y,
            z : atom.z
          }
        });
      }
    }
    viewer.render();
  }
  function MolRecenter()
  {
    viewer.zoomTo(); 
  }
  //下载
  function MolDownload(){//?
    let filename = name.value+"."+file_type.value;
    let filecontent = content.value;
    download(filename, filecontent);
  }
  function download(filename, content) {
      var blob = new Blob([content], {type: 'text/plain'});
      var url = window.URL.createObjectURL(blob);
      var a = document.createElement('a');
  
      a.style = "display: none";
      a.href = url;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
  
      setTimeout(function () {
          document.body.removeChild(a);
          window.URL.revokeObjectURL(url);
      }, 5);
  }

// el-menu
function onSelectMenu(menuIndex) {
  selectedMenu.value = menuIndex;
}
</script>
  
  
<style>
  .title{
    color: grey;
    font-size: 30px;  /* 设置字体大小为 24 像素 */
    font-family: Arial, sans-serif;
    text-align: left;  /* 设置文本居中对齐 */
    margin:0;
    margin-top: 13px;
    margin-left: 10px;
    padding:0;
  }
  /* 去除表格线 上边框 下边框 */
  .para-table >>> .el-table__row>td{
    border: none;
  }
  .para-table >>> .el-table th.is-leaf {
    border: none;
  }
  /* 分子可视化 */
  .mol-container {
    height: 500px;
    width: 100%;
    margin: 0; 
    padding: 0; 
    border: 0;
    position: relative;
  }
  /* 按钮 */
  .btn{
    width: 100%;
  }
  .el-tabs__header {
    margin-bottom: 0 ;
  }
  .custom-label {
    width:150px;
  }
  .custom-label_rd {
    width:150px;
  }
  .custom-label_hun {
    width:150px;
  }

  /* 自定义ElMenu样式 */
  .customElMenu .el-menu-item, .customElMenu .el-sub-menu {
    padding: 0 5px;
  }
  .customElMenu .el-sub-menu__title {
    padding: 0 20px 0 5px;
  }
  .customElMenu .el-sub-menu .el-sub-menu__icon-arrow {
    right: 0;
  }

</style>
  