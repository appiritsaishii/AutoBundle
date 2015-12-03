# AutoBundle

[![License](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Build Status](https://travis-ci.org/yatatsu/AutoBundle.svg?branch=master)](https://travis-ci.org/yatatsu/AutoBundle)
[ ![Download](https://api.bintray.com/packages/yatatsu/maven/autobundle/images/download.svg) ](https://bintray.com/yatatsu/maven/autobundle/_latestVersion)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-AutoBundle-brightgreen.svg?style=flat)](http://android-arsenal.com/details/1/2600)

AutoBundle generates boilerplate code for field binding with ``android.os.Bundle``.

## Usage

### 1. Generate builder method

AutoBundle supports these classes.

- `Activity`
- `Fragment`
- `BroadcastReceiver`
- `Service`

Here is example for Fragment. First, declare the field with `@AutoBundleField`.

```
public class ExampleFragment extends Fragment {
    // field with @AutoBundleField, must not be private/protected.
    @AutoBundleField
    String title;

    @AutoBundleField
    int exampleId;

    @AutoBundleField(required = false) // default is true
    int optionalId;
}
```

In caller section, you can use generated builder.

```
ExampleFragment fragment = ExampleFragmentAutoBundle
        .createFragmentBuilder("hello, example!", 1)
        .optionalId(2) // here is optional
        .build();
```

Helper class is named ``{YourClass}AutoBundle``.
FragmentBuilder also has method ``build(Fragment fragment)``,
so you can set bundle to existing instance.

For ``Intent``, here is builder example.

```
Intent intent = ExampleActivityAutoBundle.createIntentBuilder("hello, example!")
        .optionalName("optionalName")
        .fooList(messages)
        .build(this)
        .addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);
```

Builder class has both methods ``build(Context context)``, ``build(Intent intent)``.

### 2. Bind annotated fields

In target class,

```ExampleFragment.java
public class ExampleFragment extends DialogFragment {
    // field with @AutoBundleField, must not be private/protected.
    @AutoBundleField
    String title;

    @AutoBundleField
    int exampleId;

    @AutoBundleField(required = false) // default is true
    int optionalId;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AutoBundle.bind(this);
    }
}
```

Call ``AutoBundle#bind`` in ``onCreate`` and bind value to field with ``@AutoBundleField``.

In ``Intent`` case, call these method.

- ``bind(Object target, Intent intent)``
- ``bind(Object target, Bundle bundle)``
- ``bind(Activity target)`` (equals to ``bind(activity, activity.getIntent())``
- ``bind(Object target)`` (equals to ``bind(fragment, fragment.getArguments())``

``AutoBundle`` use reflection at the first time, to find generated class ``AutoBundleBindingDispatcher``,
and call internal binding method for each classes.
(This idea from [sockeqwe/FragmentArgs](https://github.com/sockeqwe/fragmentargs).)

### Advanced

#### Set key name

``key`` is key for ``Bundle``. Default is field name.
You cannot define duplicate key in one class.

```
@AutoBundleField(key = "exampleId")
int id;
```

#### Required

``required`` option is true by default.
If ``false``, Builder class has method which named key name, instead as contructor argument.

```
@AutoBundleField(required = false)
int optionalId;
```

then,

```
ExampleFragment fragment = ExampleFragmentAutoBundle
        .createFragmentBuilder()
        .optionalId(2)
        .build();
```

#### CustomConverter

``converter`` option provide custom converter for storing to bundle.
You can specify class which implements ``AutoBundleConverter<T, U>``.

```
public class ExampleFragment extends Fragment {

    @AutoBundleField(converter = DateArgConverter.class)
    Date targetDate;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AutoBundle.bind(this);
    }

    public static class DateArgConverter implements AutoBundleConverter<Date, Long> {

        @Override
        public Long convert(Date o) {
            return o.getTime();
        }

        @Override
        public Date original(Long s) {
            return new Date(s);
        }
     }
}
```

#### Restore from bundle

AutoBundle bindings are also useful as restoring value in ``onSaveInstanceState(Bundle outState)``.

```
@Override
public void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    AutoBundle.pack(this, outState);
}
```

``pack(Object object, Bundle bundle)`` stores field value to bundle.
For example, store in ``onSaveInstanceState`` and restore in ``onCreate``.

For more infomation or usage, see the sample application!

### Principle

Both Fragment and Intent are able to pass value by storing Bundle.

There is a good pattern, known as "createIntent" or "newInstance".
Static method in called class (e.x. Fragment, Activity) promised the expected data will be passed.
AutBundle provide this pattern as builder method instead of "createIntent".

## Download

AutoBundle uses annotation processing, so you need to add android-apt plugin.

```
buildscript {
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

apply plugin: 'android-apt'

dependencies {
    compile 'com.github.yatatsu:autobundle:1.0.0-beta1'
    apt 'com.github.yatatsu:autobundle-processor:1.0.0-beta1'
}
```

## Props

- [sockeqwe/FragmentArgs](https://github.com/sockeqwe/fragmentargs)
- [emilsjolander/IntentBuilder](https://github.com/emilsjolander/IntentBuilder)

Actually, AutoBundle is that just integrates above both great libraries.

## License

```
Copyright 2015 KITAGAWA, Tatsuya

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
