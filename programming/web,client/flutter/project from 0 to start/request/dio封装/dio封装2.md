<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2024-04-06 13:52:08
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2024-04-06 16:48:06
 * @FilePath: \studys\programming\web,client\flutter\project from 0 to start\request\dio封装\dio封装2.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<https://blog.csdn.net/sinat_17775997/article/details/109803193>\
- [HttpManager的定义](#httpmanager的定义)
- [响应基类](#响应基类)
- [Api的封装](#api的封装)
- [公共参数和加密等](#公共参数和加密等)
- [地址的配置](#地址的配置)
- [响应拦截器](#响应拦截器)
- [日志拦截器](#日志拦截器)
- [示例请求](#示例请求)

### HttpManager的定义
```dart
import 'package:dio/dio.dart';
import 'package:flutter_net/code.dart';
import 'package:flutter_net/dio_log_interceptor.dart';
import 'package:flutter_net/loading_utils.dart';
import 'response_interceptor.dart';
import 'result_data.dart';
import 'address.dart';
 
class HttpManager {
  static HttpManager _instance = HttpManager._internal();
  Dio _dio;
 
  static const CODE_SUCCESS = 200;
  static const CODE_TIME_OUT = -1;
 
  factory HttpManager() => _instance;
 
  ///通用全局单例，第一次使用时初始化
  HttpManager._internal({String baseUrl}) {
    if (null == _dio) {
      _dio = new Dio(
          new BaseOptions(baseUrl: Address.BASE_URL, connectTimeout: 15000));
      _dio.interceptors.add(new DioLogInterceptor());
//      _dio.interceptors.add(new PrettyDioLogger());
      _dio.interceptors.add(new ResponseInterceptors());
    }
  }
 
  static HttpManager getInstance({String baseUrl}) {
    if (baseUrl == null) {
      return _instance._normal();
    } else {
      return _instance._baseUrl(baseUrl);
    }
  }
 
  //用于指定特定域名
  HttpManager _baseUrl(String baseUrl) {
    if (_dio != null) {
      _dio.options.baseUrl = baseUrl;
    }
    return this;
  }
 
  //一般请求，默认域名
  HttpManager _normal() {
    if (_dio != null) {
      if (_dio.options.baseUrl != Address.BASE_URL) {
        _dio.options.baseUrl = Address.BASE_URL;
      }
    }
    return this;
  }
 
  ///通用的GET请求
  get(api, {params, withLoading = true}) async {
    if (withLoading) {
      LoadingUtils.show();
    }
 
    Response response;
    try {
      response = await _dio.get(api, queryParameters: params);
      if (withLoading) {
        LoadingUtils.dismiss();
      }
    } on DioError catch (e) {
      if (withLoading) {
        LoadingUtils.dismiss();
      }
      return resultError(e);
    }
 
    if (response.data is DioError) {
      return resultError(response.data['code']);
    }
 
    return response.data;
  }
 
  ///通用的POST请求
  post(api, {params, withLoading = true}) async {
    if (withLoading) {
      LoadingUtils.show();
    }
 
    Response response;
 
    try {
      response = await _dio.post(api, data: params);
      if (withLoading) {
        LoadingUtils.dismiss();
      }
    } on DioError catch (e) {
      if (withLoading) {
        LoadingUtils.dismiss();
      }
      return resultError(e);
    }
 
    if (response.data is DioError) {
      return resultError(response.data['code']);
    }
 
    return response.data;
  }
}
 
ResultData resultError(DioError e) {
  Response errorResponse;
  if (e.response != null) {
    errorResponse = e.response;
  } else {
    errorResponse = new Response(statusCode: 666);
  }
  if (e.type == DioErrorType.CONNECT_TIMEOUT ||
      e.type == DioErrorType.RECEIVE_TIMEOUT) {
    errorResponse.statusCode = Code.NETWORK_TIMEOUT;
  }
  return new ResultData(
      errorResponse.statusMessage, false, errorResponse.statusCode);
}
 
```
### 响应基类
* 默认200的情况isSuccess为true，响应为response.data,赋值给data

```dart
class ResultData {
  var data;
  bool isSuccess;
  int code;
  var headers;
 
  ResultData(this.data, this.isSuccess, this.code, {this.headers});
}
```
### Api的封装
```dart
class Api {
  ///示例请求
  static request(String param) {
    var params = DataHelper.getBaseMap();
    params['param'] = param;
    return HttpManager.getInstance().get(Address.TEST_API, params);
  }
}

```
### 公共参数和加密等
```dart
class DataHelper{
  static SplayTreeMap getBaseMap() {
    var map = new SplayTreeMap<String, dynamic>();
    map["platform"] = AppConstants.PLATFORM;
    map["system"] = AppConstants.SYSTEM;
    map["channel"] = AppConstants.CHANNEL;
    map["time"] = new DateTime.now().millisecondsSinceEpoch.toString();
    return map;
  }
  
  static string2MD5(String data) {
    var content = new Utf8Encoder().convert(data);
    var digest = md5.convert(content);
    return hex.encode(digest.bytes);
  }
}
```
### 地址的配置
```dart
class Address {
  static const String TEST_API =  "test_api";
}
```
### 响应拦截器
```dart
import 'package:dio/dio.dart';
import 'package:exchange_flutter/common/net/code.dart';
import 'package:flutter/material.dart';
import '../result_data.dart';
 
class ResponseInterceptors extends InterceptorsWrapper {
  @override
  onResponse(Response response) {
    RequestOptions option = response.request;
    try {
      if (option.contentType != null &&
          option.contentType.primaryType == "text") {
        return new ResultData(response.data, true, Code.SUCCESS);
      }
      ///一般只需要处理200的情况，300、400、500保留错误信息
      if (response.statusCode == 200 || response.statusCode == 201) {
        int code = response.data["code"];
        if (code == 0) {
          return new ResultData(response.data, true, Code.SUCCESS,
              headers: response.headers);
        } else if (code == 100006 || code == 100007) {
 
        } else {
          Fluttertoast.showToast(msg: response.data["msg"]);
          return new ResultData(response.data, false, Code.SUCCESS,
              headers: response.headers);
        }
      }
    } catch (e) {
      print(e.toString() + option.path);
 
      return new ResultData(response.data, false, response.statusCode,
          headers: response.headers);
    }
 
    return new ResultData(response.data, false, response.statusCode,
        headers: response.headers);
  }
}
 
```
### 日志拦截器
```dart
 
import 'package:dio/dio.dart';
 
///日志拦截器
class DioLogInterceptor extends Interceptor {
  @override
  Future onRequest(RequestOptions options) async {
    String requestStr = "\n==================== REQUEST ====================\n"
        "- URL:\n${options.baseUrl + options.path}\n"
        "- METHOD: ${options.method}\n";
 
    requestStr += "- HEADER:\n${options.headers.mapToStructureString()}\n";
 
    final data = options.data;
    if (data != null) {
      if (data is Map)
        requestStr += "- BODY:\n${data.mapToStructureString()}\n";
      else if (data is FormData) {
        final formDataMap = Map()
          ..addEntries(data.fields)
          ..addEntries(data.files);
        requestStr += "- BODY:\n${formDataMap.mapToStructureString()}\n";
      } else
        requestStr += "- BODY:\n${data.toString()}\n";
    }
    print(requestStr);
    return options;
  }
 
  @override
  Future onError(DioError err) async {
    String errorStr = "\n==================== RESPONSE ====================\n"
        "- URL:\n${err.request.baseUrl + err.request.path}\n"
        "- METHOD: ${err.request.method}\n";
 
    errorStr +=
        "- HEADER:\n${err.response.headers.map.mapToStructureString()}\n";
    if (err.response != null && err.response.data != null) {
      print('╔ ${err.type.toString()}');
      errorStr += "- ERROR:\n${_parseResponse(err.response)}\n";
    } else {
      errorStr += "- ERRORTYPE: ${err.type}\n";
      errorStr += "- MSG: ${err.message}\n";
    }
    print(errorStr);
    return err;
  }
 
  @override
  Future onResponse(Response response) async {
    String responseStr =
        "\n==================== RESPONSE ====================\n"
        "- URL:\n${response.request.uri}\n";
    responseStr += "- HEADER:\n{";
    response.headers.forEach(
        (key, list) => responseStr += "\n  " + "\"$key\" : \"$list\",");
    responseStr += "\n}\n";
    responseStr += "- STATUS: ${response.statusCode}\n";
 
    if (response.data != null) {
      responseStr += "- BODY:\n ${_parseResponse(response)}";
    }
    printWrapped(responseStr);
    return response;
  }
 
  void printWrapped(String text) {
    final pattern = new RegExp('.{1,800}'); // 800 is the size of each chunk
    pattern.allMatches(text).forEach((match) => print(match.group(0)));
  }
 
  String _parseResponse(Response response) {
    String responseStr = "";
    var data = response.data;
    if (data is Map)
      responseStr += data.mapToStructureString();
    else if (data is List)
      responseStr += data.listToStructureString();
    else
      responseStr += response.data.toString();
 
    return responseStr;
  }
}
 
extension Map2StringEx on Map {
  String mapToStructureString({int indentation = 2}) {
    String result = "";
    String indentationStr = " " * indentation;
    if (true) {
      result += "{";
      this.forEach((key, value) {
        if (value is Map) {
          var temp = value.mapToStructureString(indentation: indentation + 2);
          result += "\n$indentationStr" + "\"$key\" : $temp,";
        } else if (value is List) {
          result += "\n$indentationStr" +
              "\"$key\" : ${value.listToStructureString(indentation: indentation + 2)},";
        } else {
          result += "\n$indentationStr" + "\"$key\" : \"$value\",";
        }
      });
      result = result.substring(0, result.length - 1);
      result += indentation == 2 ? "\n}" : "\n${" " * (indentation - 1)}}";
    }
 
    return result;
  }
}
 
extension List2StringEx on List {
  String listToStructureString({int indentation = 2}) {
    String result = "";
    String indentationStr = " " * indentation;
    if (true) {
      result += "$indentationStr[";
      this.forEach((value) {
        if (value is Map) {
          var temp = value.mapToStructureString(indentation: indentation + 2);
          result += "\n$indentationStr" + "\"$temp\",";
        } else if (value is List) {
          result += value.listToStructureString(indentation: indentation + 2);
        } else {
          result += "\n$indentationStr" + "\"$value\",";
        }
      });
      result = result.substring(0, result.length - 1);
      result += "\n$indentationStr]";
    }
 
    return result;
  }
}
 
 
```
### 示例请求
```dart
void request() async {
    ResultData res = await Api.request("param");
    if (res.isSuccess) {
       //拿到res.data就可以进行Json解析了，这里一般用来构造实体类
            TestBean bean = TestBean.fromMap(res.data);
 
    }else{
      //处理错误
    }
  }
```