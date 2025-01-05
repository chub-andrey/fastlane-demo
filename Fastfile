default_platform(:ios)

platform :ios do

  slack_hook = "https://hooks.slack.com/services/....."

  desc "Setup Project in Apple system"
  lane :register_app do
    produce(
      username: "appleid.email@gmail.com",
      app_identifier: "com.app.identifier",
      app_name: "AppName",
      team_name: "TeamName",
      itc_team_name: "AppstoreConnectTeamName"
    )
  end


  # To begin use match, it must be seted up with command - fastlane match init
  # To be sure that Matchfile updated

  desc "Sync Code-Signing assets"
  lane :sync_signing_assets do |options|
    selectedType = options[:type]
    match(type: selectedType)
  end

  desc "Register Devices on the Developer Portal"
  lane :sync_devices_info do
    register_devices(
      devices_file: "fastlane/Devicefile"
    )
  end

  desc "Update certificates and PP"
  lane :bootstrap_code_sign do
    sync_devices_info
    sync_signing_assets(type: "development")
    sync_signing_assets(type: "adhoc")
    sync_signing_assets(type: "appstore")
  end


  # To begin use gym, it must be seted up with command - fastlane gym init
  # To be sure that Gymfile updated

  desc "Build App Store build"
  lane :build_appstore do
    run_swiftlint(mode: "lint")
    precheck_version
    sync_signing_assets(type: "appstore")
    increment_build_number
    gym(
      output_directory: "build_Appstore",
      export_method: "app-store"
    )
  end

  desc "Build Ad Hoc build"
  lane :build_adhoc do
    run_swiftlint(mode: "autocorrect")
    precheck_version
    sync_signing_assets(type: "adhoc")
    run_unit_tests
    increment_build_number
    gym(
      output_directory: "build_AdHoc",
      export_method: "ad-hoc"
    )
  end

  desc "Distribute build to AppStore"
  lane :distribute_to_appstore do
    build_appstore
    pilot(
      team_name: "TeamName",
      changelog: "Version #{lane_context[SharedValues::VERSION_NUMBER]}, Build #{lane_context[SharedValues::BUILD_NUMBER]}"
    )
   send_slack(
     message: "Distribute build to AppStore",
     version: lane_context[SharedValues::VERSION_NUMBER], 
     build: lane_context[SharedValues::BUILD_NUMBER]
   )
  end

  desc "Check version for Appstore review with Precheck tool"
  lane :precheck_version do
    precheck
    UI.success "Version meet requirements and ready for review"
  end

  # To begin use scan, it must be seted up with command - fastlane scan init
  # To be sure that Scanfile updated
  # Might be need to edit Test Scheme - in Build section check Run

  desc "Run Unit tests"
  lane :run_unit_tests do
    scan
  end

  lane :run_swiftlint do |options|
    selectedMode = options[:mode]
    
    if selectedMode == "lint"
      swiftlint(
        mode: :lint,
        config_file: ".swiftlint.yml",
        output_file: "swiftlintOutput.txt",
        ignore_exit_status: false
      )
    else
      swiftlint(
        mode: :autocorrect,
        config_file: ".swiftlint.yml",
        output_file: "swiftlintOutput.txt",
        ignore_exit_status: false
      )
    end
  end

  private_lane :send_slack do |options|
     messageText = options[:message] || "Default message"
     versionNumber = options[:version]
     buildNumber = options[:build]
     pay = options[:payload]
     isSuccess = options[:success] || false
     dat = sh("date -u")

     if buildNumber.empty?
       slack(
         message: messageText,
         payload: pay,
         default_payloads: [],
         success: isSuccess,
         slack_url: slack_hook
       )
     elsif pay.empty? && isSuccess
       slack(
         payload: {"FASTLANE PRODUCTION - #{versionNumber} (#{buildNumber})" => dat},
         slack_url: slack_hook
       )
     else
       slack(
         message: messageText,
         payload: pay,
         success: isSuccess,
         slack_url: slack_hook
       )
     end
   end

end