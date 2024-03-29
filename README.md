
```kts
 getByName("debug") {
            isTestCoverageEnabled = true
        }
        
 
apply {
    from("./jacoco.gradle")
}
```
jacoco.gradle  
```groovy
apply plugin: "jacoco"

jacoco {
    toolVersion = "0.8.4"
}
project.android.libraryVariants.all { variant ->
    def variantName = variant.name.capitalize() //ex. ProdDebug
    def realVariantName = variant.name //ex. prodDebug

    if (variant.buildType.name != "debug") {
        return
    }

    task("jacoco${variantName}TestReport", type: JacocoReport) {
        //AndroidTest後にUnitTestの内容をマージします。
        dependsOn "create${variantName}CoverageReport"
        dependsOn "test${variantName}UnitTest"

        group = "testing"
        description = "Generate Jacoco coverage reports for ${realVariantName}"

        reports {
            xml.enabled = false
            html.enabled = true
        }

        //無視するファイル(excludes)の設定を行います
        def fileFilter = ['**/R.class',
                          '**/R$*.class',
                          '**/DataBindingInfo.class',
                          '**/DataBinder*.class',
                          '**/BR.class',
                          '**/BuildConfig.*',
                          '**/Manifest*.*',
                          'android/**/*.*',
                          'androidx/**/*.*',
                          '**/Lambda$*.class',
                          '**/Lambda.class',
                          '**/*Lambda.class',
                          '**/*Lambda*.class',
                          '**/*Lambda*.*',
                          '**/*Builder.*'
        ]
        def javaDebugTree = fileTree(dir: "${buildDir}/intermediates/javac/${realVariantName}/compile${variantName}JavaWithJavac/classes", excludes: fileFilter)
        def kotlinDebugTree = fileTree(dir: "${buildDir}/tmp/kotlin-classes/${realVariantName}", excludes: fileFilter)

        def mainSrc = "${project.projectDir}/src/main/java"

        getSourceDirectories().setFrom(files([mainSrc]))
        //Java, Kotlin混在ファイル対応
        getClassDirectories().setFrom(files([javaDebugTree, kotlinDebugTree]))
        getExecutionData().setFrom(fileTree(dir: project.projectDir, includes: [
                '**/*.exec',    //JUnit Test Result
                '**/*.ec'])     //Espresso Test Result
        )
    }
} 
```
