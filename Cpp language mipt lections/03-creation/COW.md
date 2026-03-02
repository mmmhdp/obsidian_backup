`Copy-On-Write`
Is an optimization technique where multiple objects share the same data until one of them modifies it. Only when a modification happens is a real copy made.

# `Why`?
To avoid unnecessary copying and improve performance.
```cpp
class String {
    struct Data {
        std::string value;
        int ref_count;
    };

    Data* data;

public:
    String(const std::string& s) {
        data = new Data{s, 1};
    }

    String(const String& other) {
        data = other.data;
        ++data->ref_count;
    }

    void set_char(size_t i, char c) {
        if (data->ref_count > 1) {
            --data->ref_count;
            data = new Data{data->value, 1};
        }
        data->value[i] = c;
    }
};
```

