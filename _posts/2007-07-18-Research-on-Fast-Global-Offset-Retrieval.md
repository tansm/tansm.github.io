# 快速获取全局偏移量的研究

> 原文：https://www.cnblogs.com/tansm/archive/2007/07/18/822203.html

自己研究偏移时的成果,在此备份,所以难得解释。

```
public abstract class DcmsCharacteristic {

        private static int _offset = 0;
        public static int Offset {
            get {
                return _offset;
            }
        }

        public static int GetOffset(int count) {
            int result = _offset;
            _offset += count;
            return result;
        }
    }

    public static class DcmsCommonUICharacteristic  {

        static DcmsCommonUICharacteristic() {
            int offset = DcmsCharacteristic.GetOffset(1);
            DocumentAddressCommandService = offset;
        }

        public static readonly int DocumentAddressCommandService;
    }
```

使用非常方便

```
int a = DcmsCommonUICharacteristic.DocumentAddressCommandService;
```
