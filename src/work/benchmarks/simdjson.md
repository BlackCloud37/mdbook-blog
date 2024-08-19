# simdjson

## `find_field` 方式对比

在 `simdjson::ondemand` 模型里, 可以通过 `find_field`, `find_field_unordered` 或 `operator[]` 来查字段, `find_field` 要求有序访问, 后两者则可以无序

简单测试性能

### 结果(O2)
```
|               ns/op |                op/s |    err% |     total | benchmark
|--------------------:|--------------------:|--------:|----------:|:----------
|              118.30 |        8,452,987.49 |    0.6% |      0.01 | `find_in_order`
|              127.27 |        7,857,332.54 |    1.0% |      0.01 | `find_unordered_in_order`
|              217.35 |        4,600,790.26 |    0.5% |      0.01 | `find_unordered_unordered`
|              130.00 |        7,692,027.51 |    1.2% |      0.01 | `operator [] in order`
|              219.08 |        4,564,583.22 |    0.8% |      0.01 | `operator [] unordered`
```

### 结论
- 在有序访问的前提下, `find_field` 最快, 但和 `find_field_unordered` 差距不大
- 无序访问导致 `rewind` 后开销会近乎翻倍(大概就是重新遍历的代价)
- `operator[]` 和 `find_field_unordered` 差不多

### 代码
```cpp
#define ANKERL_NANOBENCH_IMPLEMENT
#include "nanobench.h"
#include "simdjson.h"

auto json = R"({"e":"ORDER_TRADE_UPDATE","T":1723542730357,"E":1723542730357,"o":{"s":"ADAUSDT","c":"15570553482120512","S":"BUY","o":"LIMIT","f":"GTX","q":"900","p":"0.33210","ap":"0","sp":"0","x":"CANCELED","X":"CANCELED","i":43631626214,"l":"0","z":"0","L":"0","n":"0","N":"USDT","T":1723542730357,"t":0,"b":"598.14520","a":"301.11419","m":false,"R":false,"wt":"CONTRACT_PRICE","ot":"LIMIT","ps":"BOTH","cp":false,"rp":"0","pP":false,"si":0,"ss":0,"V":"EXPIRE_MAKER","pm":"NONE","gtd":0}})"_padded;

int main()
{
    simdjson::ondemand::parser parser;

    ankerl::nanobench::Bench().run("find_in_order", [&](){
        auto doc = parser.iterate(json);

        std::string_view e = doc.find_field("e");
        assert(e == "ORDER_TRADE_UPDATE");

        auto o = doc.find_field("o").get_object();

        std::string_view s = o.find_field("s");
        assert(s == "ADAUSDT");

        double q = o.find_field("q").get_double_in_string();
        assert(q == 900);       
    });

    ankerl::nanobench::Bench().run("find_unordered_in_order", [&](){
        auto doc = parser.iterate(json);

        std::string_view e = doc.find_field_unordered("e");
        assert(e == "ORDER_TRADE_UPDATE");

        auto o = doc.find_field_unordered("o").get_object();

        std::string_view s = o.find_field_unordered("s");
        assert(s == "ADAUSDT");

        double q = o.find_field_unordered("q").get_double_in_string();
        assert(q == 900);       
    });

    ankerl::nanobench::Bench().run("find_unordered_unordered", [&](){
        auto doc = parser.iterate(json);

        std::string_view e = doc.find_field_unordered("e");
        assert(e == "ORDER_TRADE_UPDATE");

        auto o = doc.find_field_unordered("o").get_object();

        double q = o.find_field_unordered("q").get_double_in_string();
        assert(q == 900);       

        std::string_view s = o.find_field_unordered("s");
        assert(s == "ADAUSDT");
    });

    ankerl::nanobench::Bench().run("operator [] in order", [&](){
        auto doc = parser.iterate(json);

        std::string_view e = doc["e"];
        assert(e == "ORDER_TRADE_UPDATE");

        auto o = doc["o"];

        std::string_view s = o["s"];

        double q = o["q"].get_double_in_string();

        assert(s == "ADAUSDT");

        assert(q == 900);
    });

    ankerl::nanobench::Bench().run("operator [] unordered", [&](){
        auto doc = parser.iterate(json);

        std::string_view e = doc["e"];
        assert(e == "ORDER_TRADE_UPDATE");

        auto o = doc["o"];

        double q = o["q"].get_double_in_string();

        std::string_view s = o["s"];

        assert(s == "ADAUSDT");

        assert(q == 900);
    });

    return 0;
}
```
