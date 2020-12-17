# 敏感词过滤

```java
package com.clubfactory.lm.connector.core.utils;

import org.springframework.util.CollectionUtils;

import java.util.*;

public class SensitiveWordUtil {

    public static Map genSensitiveWordToHashMap(Set<String> keyWordSet) {
        Map sensitiveWordMap = new HashMap(keyWordSet.size());
        String key = null;
        Map nowMap = null;
        Map<String, String> newWorMap = null;
        Iterator<String> iterator = keyWordSet.iterator();
        while (iterator.hasNext()) {
            key = iterator.next();
            nowMap = sensitiveWordMap;
            for (int i = 0; i < key.length(); i++) {
                char keyChar = key.charAt(i);
                Object wordMap = nowMap.get(keyChar);

                if (wordMap != null) {
                    nowMap = (Map) wordMap;
                } else {
                    //不存在则，则构建一个map，同时将isEnd设置为0，因为他不是最后一个
                    newWorMap = new HashMap<String, String>();
                    newWorMap.put("isEnd", "0");
                    nowMap.put(keyChar, newWorMap);
                    nowMap = newWorMap;
                }

                if (i == key.length() - 1) {
                    nowMap.put("isEnd", "1");
                }
            }
        }
        return sensitiveWordMap;
    }

    public static String SearchAndReplaceSensitiveWord(String txt, Map sensitiveWordMap) {
        int sensitiveWordLength = 0;
        int startIndex = 0;
        int endIndex = 0;
        char word = 0;
        Map nowMap = sensitiveWordMap;
        Set<String> sensitiveWordSet = new HashSet<>();
        char[] chars = txt.toCharArray();
        for (int i = 0; i < txt.length(); i++) {
            word = txt.charAt(i);
            nowMap = (Map) nowMap.get(word);
            if (nowMap != null) {
                sensitiveWordLength++;
                if ("1".equals(nowMap.get("isEnd"))) {
                    startIndex = i + 1 - sensitiveWordLength;
                    endIndex = i;
                    //敏感词是在文本的开头
                    if (startIndex == 0) {
                        if (i == txt.length() - 1) {
                            sensitiveWordSet.add(txt.substring(startIndex, endIndex+1));
                            nowMap = sensitiveWordMap;
                            sensitiveWordLength = 0;
                            continue;
                        } else {
                            if (Arrays.asList(' ', ',', '.').contains(chars[endIndex + 1])) {
                                sensitiveWordSet.add(txt.substring(startIndex, endIndex+1));
                                nowMap = sensitiveWordMap;
                                sensitiveWordLength = 0;
                                continue;
                            }else{
                                continue;
                            }
                        }
                    }
                    //敏感词在末尾
                    if (i == txt.length() - 1) {
                        if (chars[startIndex - 1] == ' ') {
                            sensitiveWordSet.add(txt.substring(startIndex, endIndex+1));
                            nowMap = sensitiveWordMap;
                            sensitiveWordLength = 0;
                            continue;
                        }
                    }
                    //敏感词在中间
                    if (chars[startIndex - 1] == ' ' && Arrays.asList(' ', ',', '.').contains(chars[endIndex + 1])) {
                        sensitiveWordSet.add(txt.substring(startIndex, endIndex+1));
                        nowMap = sensitiveWordMap;
                        sensitiveWordLength = 0;
                        continue;
                    }
                }
            }
            else {     //不存在，直接返回
                nowMap = sensitiveWordMap;
                sensitiveWordLength = 0;
            }
        }
        if (CollectionUtils.isEmpty(sensitiveWordSet)) {
            return txt;
        } else {
            for (String sensitiveWord : sensitiveWordSet) {
                txt = txt.replace(sensitiveWord, "");
            }
        }
        return txt;
    }


    public static void main(String[] args) {
        List<String> list = Arrays.asList("push daggers",
                "health",
                "pepper spray",
                "nunchakus",
                "spring batons",
                "filter",
                "crossbow",
                "knive",
                "knife",
                "stun gun",
                "bullpups",
                "knuckle",
                "shuriken",
                "gift",
                "flashlight",
                "bullpup",
                "fuel filter",
                "push dagger",
                "cartridge magazine",
                "cartridge magazines",
                "taser",
                "finger ring",
                "gun",
                "blowguns",
                "brass knuckle",
                "crossbows",
                "kusari ",
                "manrikigusari",
                "laser",
                "firearms",
                "air conditioner",
                "mace",
                "finger rings",
                "air filter",
                "fuel filters",
                "spiked wristbands",
                "tasers",
                "spiked wristband",
                "air compressor",
                "knives",
                "brass knuckles",
                "flashlights",
                "spring baton",
                "stun guns",
                "blowgun",
                "firearm",
                "military",
                "fuel"
        );


        System.out.println(SearchAndReplaceSensitiveWord("fuel filter x.", genSensitiveWordToHashMap(new HashSet<>(list))));
    }
}
```

