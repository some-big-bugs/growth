# 字符串匹配算法 Sunny



```python
def right_step(ch: str, sub_str: str) -> int:
    offset: int = 1

    sub_str_len: int = len(sub_str)

    for index in range(sub_str_len):
        if ch == sub_str[sub_str_len - index - 1]:
            break
        offset += 1
    return offset


def index(main_str: str, sub_str: str) -> int:

    sub_str_len: int = len(sub_str)
    main_str_len: int = len(main_str)
    main_str_offset: int = 0
    allow_offset: int = main_str_len - sub_str_len

    if len(sub_str) > len(main_str):
        return -1

    while main_str_offset <= allow_offset:

        # 第一步判断是否字符串相等
        offset_index = 0
        while main_str[main_str_offset + offset_index] == sub_str[offset_index]:
            offset_index += 1
            if offset_index == sub_str_len:
                return main_str_offset

        # 不相等的情况
        if main_str_offset == allow_offset:
            # 已经到了最后一位
            return -1

        main_str_offset += right_step(main_str[main_str_offset + sub_str_len], sub_str)
    return -1
```

