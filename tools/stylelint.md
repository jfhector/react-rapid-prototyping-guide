#stylelint

### Disabling stylelint for tsx

```
       "extends": "stylelint-config-standard",
       "rules": {
             "indentation": 4
-      }
+      },
+      "ignoreFiles": ["**/*.tsx"]
 }

```