function _smartQuestionairFactory(){
    return sq= 
    {
        questionTypeToLoad:null,
        questionairTitle:null,
        questionCounter: 0,
        anwLocation: null,
        questionLocation: null,
        securityActivityLocation: null,
        payRateLocation:null,
        questionModel:null,
        theAnswerModel:[],
        rAWAnw:[],
        payRate: null,
        DOMLocation: null,
        addUpAnwser:null,
        getFirstQuestion:function(){
                for(var i = 0; i < this.questionModel.length;i++){
                                if(this.questionModel[i].IsFirstQuestion == true){
                                                return i;
                                }
                }
                return 0;
        },
        extendedAnwFactory: function(data){return this._answerModelFactory(); },
        initQuestionair:function(){
            if (getParameterByName("doTest") == "true") {
                tester();
            } else {
                var instanceOfMySelf  = this;
                this.loadQuestions(this.questionLocation, this.questionTypeToLoad, function (questionArray) {
                    instanceOfMySelf.questionModel = questionArray;
                    instanceOfMySelf.loadAnwsers(instanceOfMySelf.anwLocation, function(anwsers){
                                 for (var i = 0; i < anwsers.length; i++) {
                                                                var myQues =  instanceOfMySelf.findQuestion(instanceOfMySelf.questionModel, anwsers[i].myQuestionId);
                                                                if( myQues != null){
                                                                myQues.answer.push(anwsers[i]);
                                                                }
                                                }
                                                
                                                instanceOfMySelf.qControl(instanceOfMySelf.questionModel, instanceOfMySelf.questionModel[instanceOfMySelf.getFirstQuestion()], instanceOfMySelf.DOMLocation);
                    });
                });
            }
        },
        getQuestionHTML: function(q, qNumber){
            var anws = "";
            if (q.type == "PreDefined") {
                anws = "<select id='preDefinedSel'><option>Please Select</option>";
                for (var i = 0; i < q.answer.length; i++) {
                    anws += "<option value=\"" + i + "\">" + q.answer[i].answerText + "</option>";
                }
                anws += "</select>";

            } else if (q.type == "UserDefined") {
                anws = q.answer[0].myHTMLCtn();
            } else if (q.type =  "UserDefined With PreDefined"){
                 anws = "<select id='preDefinedSel'><option>Please Select</option>";
                for (var i = 0; i < q.answer.length; i++) {
                    anws += "<option value=\"" + i + "\">" + q.answer[i].answerText + "</option>";
                }
                anws += "</select>";
                                                                anws += q.answer[0].myHTMLCtn();

            }
            var ctnSetup = "<span>"+this.questionairTitle+"</span><fieldset style='border: 1px solid blue; border-radius:25px;'><legend>Question " + qNumber;
            var endLgnd = "</legend><span>" + q.text + "</span><br/>" + anws + "<input id='currQID'type='hidden' value='" + q.qId + "' /><br/><button id='nextQuestionCtn'>Next Question</button>&nbsp<button id='cancelCtn' >Cancel</button>";
            return ctnSetup + endLgnd;
        },
        qControl:function(questions, question, locationForContent){
            this.questionCounter++;
            var ctn = this.getQuestionHTML(question, this.questionCounter);
            $("#" + locationForContent).html(ctn);
            var instanceOfMyself = this;
            $("#nextQuestionCtn").click(function () {
                var currentQId = $("#currQID").val();
                var anwIndex = 0;
                if (question.type != "UserDefined") {
                    anwIndex = $("#preDefinedSel").val();
                }
                var myQ = instanceOfMyself.findQuestion(questions, currentQId);
                var questionairIsComplete = false;
                                if (myQ.answer[parseInt(anwIndex)].nextQId == -1 || myQ.answer[parseInt(anwIndex)].nextQId == null) {
                                    if(true == instanceOfMyself.pushCalculatedAnwser(myQ.answer[parseInt(anwIndex)],instanceOfMyself)){           
                                    questionairIsComplete =true;
                                    instanceOfMyself.addUpAnwser();
                                 }else{
                                                this.questionCoutner--;
                                                instanceOfMyself.qControl(questions, question, locationForContent);
                                 }
            
                } else {
                    if(true == instanceOfMyself.pushCalculatedAnwser(myQ.answer[parseInt(anwIndex)],instanceOfMyself)){
                                    var newCTN = instanceOfMyself.findQuestion(questions, myQ.answer[parseInt(anwIndex)].nextQId);
                                    if(newCTN == null){
                                       questionairIsComplete = true;
                                       instanceOfMyself.addUpAnwser();
                                    }
                    }else{
                                this.questionCounter--;
                                instanceOfMyself.qControl(questions, question, locationForContent);
                    }
                }
                if(questionairIsComplete == true){
                                instanceOfMyself.qControl(questions, newCTN, locationForContent);
                }
            });
        },
        pushCalculatedAnwser:function(anw,myInstance){
            var myQ = this.findQuestion(myInstance.questionModel, anw.myQuestionId);
            if (myQ.type.indexOf("UserDefined")  > -1 ) {
                return myQ.answer[0].getAnwFromHTML(anw,this);

            } else {
                var theAnwIndex = $("#preDefinedSel").val();
                if (myQ.purpose == "Scope") {
                    var rAW = myInstance._realizedAnwFactory();
                    rAW.valueType = "Scope";
                    rAW.value = myQ.answer[theAnwIndex].valueFun();
                    myInstance.rAWAnw.push(rAW);
                } else {
                    var foundScopedAnw = -1;
                    for (var i = myInstance.rAWAnw.length-1; i >= 0 ; i--) {
                        if (myInstance.rAWAnw[i].valueType == "Scope") {
                            foundScopedAnw = i;
                            i = -1;
                        }
                    }
           
                    this.calculatePreDefinedAnwser(myQ.answer[theAnwIndex], myInstance.securityActivityLocation, myInstance.rAWAnw[foundScopedAnw], function (myData2) {
                        var rAW = myInstance._realizedAnwFactory();
                        rAW.valueType = "Value";
                        rAW.value = myData2.value;
                        rAW.nonValueAnwsers = myData2.nonValueAnwsers 
                        myInstance.rAWAnw.push(rAW);
                    });
             
                }
            }
            return true;
        },
        calculatePreDefinedAnwser:function(anwser , loc, scopeAnw, callBack){
                var causeIneedAStatic_ThisIsWrong = _smartQuestionairFactory();
            var myrealAnw =  causeIneedAStatic_ThisIsWrong._realizedAnwFactory();
            if(anwser.activity1 != null ){
                $.getJSON(loc + "(" + anwser.activity1+")", function (data) {
                        
                    var thing = data.d;
                        
                    for (t in thing) {
                        if (t.indexOf(scopeAnw.value) > -1 && thing[t] != null) {
                            myrealAnw.value = thing[t];
                        }        
                    }
                    myrealAnw.valueType = thing.valueTypeValue;
                    myrealAnw.nonValueAnwsers = thing.Activity;
                    callBack(myrealAnw);
                        
                
                }).fail(function(){alert("error getting stuff")});
            }
            var myrealAnw2 =  causeIneedAStatic_ThisIsWrong._realizedAnwFactory();

            if(anwser.activity2 != null ){
                $.getJSON(loc + "(" + anwser.activity2+")", function (data) {
                        
                    var thing = data.d;
                        
                    for (t in thing) {
                        if (t.indexOf(scopeAnw.value) > -1 && thing[t] != null) {
                            myrealAnw2.value = thing[t]; 
                        }        
                    }
                    myrealAnw2.valueType = thing.valueTypeValue;
                    myrealAnw2.nonValueAnwsers = thing.Activity;

                    callBack(myrealAnw2);
                        
                
                }).fail(function(){alert("error getting stuff")});
            }
        },
        findQuestion:function(qS, idToFind){
            for (var i = 0; i < qS.length; i++) {
                if (qS[i].qId == idToFind) {
                    return qS[i];
                }

            }
            return null;
        },
        loadQuestions:function(restSystemLocation,what ,successCallback){
            var instanceOfMySelf = this;
            $.getJSON(restSystemLocation, {}, function (data) {
                var modelArray = new Array();
                for (var c = 0; c < data.d.results.length; c++) {
                    var tempHoldId = data.d.results[c].Id;
                    //var et = data.da.results._metadata.etag;
                    var myQuestionairs = data.d.results[c].Questionair.split(",");
                    for(var i = 0; i < myQuestionairs.length; i++){
                        if(myQuestionairs[i].toLowerCase() == what.toLowerCase()){
                            var qu = instanceOfMySelf._questionModelFactory();         
                            qu.qId = data.d.results[c].Id;
                            qu.type = data.d.results[c].QuestionTypeValue;
                            qu.text = data.d.results[c].QuestionText;
                            qu.purpose = data.d.results[c].PurposeValue;
                            qu.IsFirstQuestion = data.d.results[c].IsFirstQuestion;
                            modelArray.push(qu);
                        }
                    }

                }
                successCallback(modelArray);

            });
        },
        loadAnwsers:function(restSystemLocation, successCallback){
            var instanceOfMyself = this;
            $.getJSON(restSystemLocation, {}, function (data) {
                var modelArray = new Array();
                for (var c = 0; c < data.d.results.length; c++) {
                    var tempHoldId = data.d.results[c].Id;
                    //var et = data.da.results._metadata.etag;
                    var qu = instanceOfMyself.extendedAnwFactory(data);                    
                    qu.answerText = data.d.results[c].AnswerText;
                    qu.myValue = data.d.results[c].AnswerText;
                    qu.activity1 = data.d.results[c].SecurityActivityOneId;
                    qu.activity2 = data.d.results[c].SecurityActivityTwoId;
                    qu.nextQId = data.d.results[c].NextQuestionId;
                    qu.myQuestionId = data.d.results[c].MyQuestionId;
                    qu.type = data.d.results[c].AnswerType;
                    
                    modelArray.push(qu);


                }
                successCallback(modelArray,instanceOfMyself);

            });
        },
        _questionModelFactory:function() {
            return { qId: -1, type: null, text: null, answer: [], purpose: null,IsFirstQuestion:null };
        },
        _answerModelFactory:function() {
            return {
                answerText: null,
                activity1: -1,
                activity2: -1,
                valueFun: function () { return this.myValue; },
                nextQId: -1,
                category: null,
                myQuestionId: -1,
                myValue: null,
                type: null,
                myHTMLCtn: null,
                getAnwFromHTML: null
            };
        },
        _realizedAnwFactory:function(){
            return { anwHTML: null, value: null, valuefun: null, valueType: null, qId: -1, nonValueAnwsers: null };
        }
    
    }
}
