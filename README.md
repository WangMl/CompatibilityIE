# Chrome兼容IE
针对比较典型的几类情况

 **事件兼容**
code如下
```javascript
   window.ct = {};
   (function(NS){
	   var EventUtil = {
	       addHandler:function(element, type, handler){
	          if(element.addEventListener){
	             element.addEventListener(type,handler,false);//Chrome支持
	          } else if(element.attachEvent){
	             element.attacheEvent("on" + type, handler);//IE9+支持
	          }else{
	             element["on" + type] = handler;//IE8-支持
	          }
	       },
	       removeHandler:function(element, type, handler){
	          if(element.removeEventListener){
	             element.removeEventListener(type, handler, false);//Chrome支持
	          }else if(element.detachEvent){
	             element.detachEvent("on" + type, handler);//IE9+支持
	          }else{
	             element["on" + type] = null;//IE8-支持
	          }
	       },
	       getEvent:function(event){
	         return event ? event : window.event;
	       },
	       getTarget:function(event){
	         return event.target || event.srcElement;
	       },
	       preventDefault:function(event){
	         if(event.preventDefault){
	            event.preventDefault();
	         }else{
	            event.returnValue = false;
	         }
	       },
	       stopPropagation:function(event){
	          if(event.stopPropagation){
	             event.stopPropagation();
	          }else{
	             event.cancelBubble = true;
	          }
	       }
	   }
	   NS.EventUtil = EventUtil;
   })(window.ct);
   
```
 **XPath兼容**
```javascript
   window.ct = {};
   (function(NS){
       var createDocument = function(){
          if(typeof arguments.callee.activeXArg != "string"){
	          var versions = ["MSXML2.DOMDocument.6.0","MSXML2.DOMDocument.3.0","MSXML2.DOMDocument"],
	              i, len;
              for(i=0, len=versions.length; i< len; i++){
                  try{
                     new ActiveXObject(versions[i]);
                     arguments.callee.activeXArg = versions[i];
                     break;                  
                  }catch(e){
                     //step
                  }
              }
          }
          return new ActiveXObject(arguments.callee.ActiveXArg);
       }

       var parseXml = function(xml){
          var xmldom = null;
          if(typeof DOMParser != "undefined"){
              xmldom = (new DOMParser()).parseFromString(xml,"text/xml");
              var errors = xmldom.getElementsByTagName("parsererror");
              if(errors.length){
                 throw new Error("XML parsing error: " + error[0].textContent);
              }
          }else if(typeof ActiveXObject != "undefined"){
              xmldom = createDocument();
              xmldom.loadXML(xml);
              if(xmldom.parseError != 0){
                 throw new Error("XML parsing error: " + xmldom.parserError.reason);
              }
          }else{
              throw new Error("No XML parser avaiable");
          }
          return xmldom;
       }
       var serializeXml = function(xmldom){
          if(typeof XMLSerializer != "undefined"){
              return (new XMLSerializer()).serialToString(xmldom);
          }else if(typeof xmldom.xml != "undefined"){
              return xmldom.xml;
          }else{
              throw new Error("Could not serialize XML DOM.");
          }
       }

       var selectSingleNode = function(context, expression, namespaces){
           var doc = (context.nodeType != 9 ? context.ownerDocument : context);
           if(typeof doc.evaluate != "undefined"){
               var nsresolver = null;
               if(namespaces instanceof Objectt){
                  nsresolver = function(prefix){
                      return namespaces[prefix];
                  }
               }
               var result = doc.evaluate(expression, context, nsresolver, XPathResult.FIRST_ORDERED_NODE_TYPE, null);
               return (result != null ? result.singleNodeValue : null);
           } 
       }

       var selectNodes = function(context, expression, namespaces){
           var doc = (context.nodeType != 9 ? context.ownerDocument : context);

           if(typeof doc.evaluate != "undefined"){
               var nsresolver = null;
               if(namespaces instanceof Object){
                    nsresolver = function(prefix){
                        return namespaces[prefix];
                    }
               }
               var result = doc.evaluate(expression, context, nsresolver, XPathResult.ORDERED_NODE_SNAPSHOT_TYPE, null);
               var nodes = new Array();
               if(result != null){
                   for(var i=0,len=result.snapshotLength;i<len;i++){
                       nodes.push(result.snapshotItem(i));
                   }
               }
               return nodes;
           }else if(typeof context.selectNodes != "undefined"){
              //创建命名空间字符串
              if(namespaces instanceof Object){
                  var ns = "";
                  for(var prefix in namespaces){
                      if(namespaces.hasOwnProperty(prefix)){
                          ns += "xmlns:" + prefix + "='" + namespaces[prefix] + "' ";
                      }
                      doc.setProperty("SelectionNamespaces",ns);
                  }
                  var result = context.selectNodes(expression);
                  var nodes = new Array();
              
                  for(var i=0,len=result.length;i<len;i++){
                      nodes.push(result[i]);
                  }
                  return nodes;
              }else{
                  throw new Error("No XPath engine found");
              }
           }
       }
       NS.CustomXML = {
           parseXml: function(xml){
              return parseXml(xml); 
           },
           serializeXml: function(xmldom){
              return serializeXml(xmldom);
           } 
       }
   })(window.ct);
```
 **XMLRequestHttp兼容**
