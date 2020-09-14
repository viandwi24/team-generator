<template>
  <Navbar />
  <div class="container my-4">
    <div class="row justify-content-center">
      <div class="col-lg-12">
        <ul class="nav nav-tabs" id="myTab" role="tablist">
          <li class="nav-item">
            <a class="nav-link active" id="source-tab" data-toggle="tab" href="#source" role="tab" aria-controls="source" aria-selected="true">Source</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" id="result-tab" data-toggle="tab" href="#result" role="tab" aria-controls="result" aria-selected="false">Result</a>
          </li>
        </ul>
        <div class="tab-content" id="myTabContent">
          <div class="tab-pane fade show active" id="source" role="tabpanel" aria-labelledby="source-tab">
            <div class="row">
              <div class="col-lg-9">
                <div class="card mt-4">
                  <div class="card-header">Team</div>
                  <div class="card-body">
                    <div>
                      <div class="custom-control custom-radio">
                        <input type="radio" class="custom-control-input" name="generateType" value="team" v-model="generateType" id="check2">
                        <label class="custom-control-label" for="check2">
                          Generate group by number of teams
                        </label>
                      </div>
                      <div class="form-group mt-2">
                        <label>Number of teams :</label>
                        <select class="form-control" v-model="generateNumber">
                          <option v-for="i in 100" :key="i" :value="i" v-text="i"></option>
                        </select>
                      </div>
                    </div>
                    <hr>
                    <div>
                      <div class="custom-control custom-radio">
                        <input type="radio" class="custom-control-input" name="generateType" value="name" v-model="generateType" id="check3">
                        <label class="custom-control-label" for="check3">
                          Generate group by number of name per team
                        </label>
                      </div>
                      <div class="form-group mt-2">
                        <label>Number of name per team :</label>
                        <select class="form-control" v-model="generateNumber">
                          <option v-for="i in 100" :key="i" :value="i" v-text="i"></option>
                        </select>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="card mt-4">
                  <div class="card-header">Data Source Text</div>
                  <div class="card-body">
                    <textarea class="form-control" rows="10" placeholder="data source..." v-model="source"></textarea>
                  </div>
                </div>
              </div>
              <div class="col-lg-3">
                <div class="card mt-4">
                  <div class="card-header">Option</div>
                  <div class="card-body">
                    <div>
                      <div>
                        <div class="custom-control custom-radio">
                          <input name="splitType" type="radio" class="custom-control-input" v-model="options.splitType" value="delimiter" id="check11">
                          <label class="custom-control-label" for="check11">Split With Delimiter</label>
                        </div>
                        <div class="form-group">
                          <label>Delimiter :</label>
                          <input v-model="options.delimiter" type="text" class="form-control form-control-sm">
                        </div>
                      </div>
                      <div>
                        <div class="custom-control custom-radio">
                          <input name="splitType" type="radio" class="custom-control-input" v-model="options.splitType" value="regex" id="check12">
                          <label class="custom-control-label" for="check12">Split With RegExp</label>
                        </div>
                        <div class="form-group">
                          <label>Regex pattern :</label>
                          <input v-model="options.regexPattern" type="text" class="form-control form-control-sm">
                        </div>
                      </div>
                    </div>
                    <hr>
                    <div class="custom-control custom-checkbox">
                      <input type="checkbox" class="custom-control-input" id="check10" v-model="options.random">
                      <label class="custom-control-label" for="check10">Random Name</label>
                    </div>
                    <div class="custom-control custom-checkbox">
                      <input type="checkbox" class="custom-control-input" id="check13" v-model="options.filterDuplicateName">
                      <label class="custom-control-label" for="check13">Filter duplicate name</label>
                    </div>
                    <button class="btn btn-sm btn-primary" @click="generateNow">Generate</button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Result tab -->
          <div class="tab-pane fade pt-4" id="result" role="tabpanel" aria-labelledby="result-tab">
            <div class="alert alert-primary" role="alert">
              {{ sourceName.length }} Total Name,
              {{ generateResult.length }} Groups Generate,
              {{ duplicateCount }} Duplicate Name Found.
              {{ duplicateRemovedCount }} Duplicate Name Removed.
            </div>
            <table v-for="(group, i) in generateResult" :key="i" class="table table-bordered">
              <tr>
                <th colspan="2">Team {{ i+1  }}</th>
              </tr>
              <tr v-for="(item, j) in group" :key="j">
                <td width="5%">{{ j+1 }}</td>
                <td>{{ capitalizeEachWords(item) }}</td>
              </tr>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import Navbar from './components/Navbar'

export default {
  name: 'App',
  components: {
    Navbar
  },
  setup () {
    const source = ref('Name 1, Name 2, Name 3')
    const sourceName = ref([])
    const duplicateCount = ref(0)
    const duplicateRemovedCount = ref(0)
    const generateType = ref('team')
    const generateNumber = ref(2)
    const generateResult = ref([])
    const options = {
      splitType: 'delimiter',
      delimiter: ', ',
      regexPattern: '\\r?\\n',
      filterDuplicateName: false,
      random: true
    }
    const { generateByNumberTeam, generateByNumberName } = useOurGenerateGroups()
    const capitalizeEachWords = (str) =>  str.replace(/\w\S*/g, function(txt){ return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase() })

    const generateNow = () => {
      let result
      let processPrepare = prepareBeforeGenerate(source.value, options)
      sourceName.value = processPrepare.value
      duplicateCount.value = processPrepare.duplicate
      duplicateRemovedCount.value = processPrepare.duplicateRemoved

      if (generateType.value == 'team') {
        result = generateByNumberTeam(
          sourceName.value,
          generateNumber.value
        )
      } else if (generateType.value == 'name') {
        result = generateByNumberName(
          sourceName.value,
          generateNumber.value
        )
      }
      generateResult.value = result
      alert('Generate success! Click Tab "Result" to view generate result.')
    }

    return {
      source,
      generateType,
      generateNumber,
      generateResult,
      options,
      generateNow,
      sourceName,
      capitalizeEachWords,
      duplicateCount,
      duplicateRemovedCount
    }
  }
}

function useOurGenerateGroups () {
  const generateByNumberName = (myArray, chunk_size) => {
    var index = 0;
    var arrayLength = myArray.length;
    var tempArray = [];
    for (index = 0; index < arrayLength; index += chunk_size) {
        let myChunk = myArray.slice(index, index+chunk_size);
        tempArray.push(myChunk);
    }
    return tempArray;
  }

  const generateByNumberTeam = (data, teamsCount) => {
    if (data.length < teamsCount) {
      alert('Number Group Generate to many!')
      return []
    }

    //
    let result = []
    let perTeam = data.length / teamsCount
    let currTeam = -1
    for (let index = 0; index < data.length; index++) {
      if (index % perTeam < 1) {
        currTeam++
        result.push([])
      }

      result[currTeam].push(data[index])
    }

    return result
  }

  return {
    generateByNumberTeam,
    generateByNumberName
  }
}

function prepareBeforeGenerate (source, options) {
  // convert to array with split type
  let sourceArr = []
  if (options.splitType == 'delimiter') {
    sourceArr = source.toLowerCase().split(options.delimiter)
  } else if (options.splitType == 'regex') {
    let reg = new RegExp(options.regexPattern, "g")
    sourceArr = source.toLowerCase().split(reg)
  }

  let result = {
    value: [...sourceArr],
    duplicate: 0,
    duplicateRemoved: 0
  }

  // random name ?
  let tempRand = result.value
  if (options.random) {
    var currentIndex = tempRand.length, temporaryValue, randomIndex;
    while (0 !== currentIndex) {
      // Pick a remaining element...
      randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex -= 1;
      temporaryValue = tempRand[currentIndex];
      tempRand[currentIndex] = tempRand[randomIndex];
      tempRand[randomIndex] = temporaryValue;
    }
  }


  // find duplicate name
  let tempSearch = []
  result.value.forEach((val) => {
    if (tempSearch.indexOf(val) === -1) {
      tempSearch.push(val)
    } else {
      result.duplicate++
    }
  })

  // remove duplicate name
  if (options.filterDuplicateName) {
    result.value = Array.from(new Set(result.value))
    result.duplicateRemoved = sourceArr .length - result.value.length
  }

  return result
}
</script>
