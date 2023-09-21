```swift
import UIKit
import Kingfisher

final class ProfileViewController: UIViewController {
  
    private let profileView = ProfileView(frame: .zero)
    private var profileImageServiceObserver: NSObjectProtocol?
    private var alertPresenter: AlertPresenterProtocol?
    private let profileService = ProfileService.shared
    private let profileImageService = ProfileImageService.shared
    private let imagesListService = ImagesListService.shared
    
    override func loadView() {
        view = profileView
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .ypBlack
        updateProfileDetails(profile: profileService.profile)
        alertPresenter = AlertPresenter(viewController: self)
    }
    
    func updateProfileDetails(profile: Profile?) {
        guard let profile = profile else {
            return
        }
        profileView.profileName?.text = profile.name
        profileView.profileLogin?.text = profile.loginName
        profileView.profileDescription?.text = profile.bio
        
        profileImageServiceObserver = NotificationCenter.default
            .addObserver(
                forName: ProfileImageService.didChangeNotification,
                object: nil,
                queue: .main
            ) { [weak self] _ in
                guard let self = self else { return }
                self.updateAvatar()
            }
        updateAvatar()
    }
    
    private func exitProfile() {
        OAuth2TokenStorage().token = nil
        WebViewViewController.clean()
        cleanService()
        
        guard let window = UIApplication.shared.windows.first else {
            return assertionFailure("Invalid Configuration")
        }
        window.rootViewController = SplashViewController()
    }
    
    private func showAlertExitProfile() {
        let model = AlertModelTwoButton(
            title: "Пока, пока!",
            message: "Уверены что хотите выйти?",
            buttonTextOne: "Да",
            buttonTextTwo: "Нет",
            completionOne: { [weak self] in
                guard let self = self else { return }
                exitProfile()
            },
            completionTwo: nil
        )
        alertPresenter?.showTwoButton(model)
    }
    
    private func cleanService() {
        profileService.cleanProfile()
        profileImageService.cleanProfileImageURL()
        imagesListService.cleanImagesList()
    }
    
    @objc
    private func didTapLogoutButton() {
        showAlertExitProfile()
    }
    
    private func updateAvatar() {
        guard
            let profileImageURL = ProfileImageService.shared.avatarURL,
            let imageURL = URL(string: profileImageURL)
        else { return }
        let processor = RoundCornerImageProcessor(cornerRadius: 61)
        profileView.profileImageView.kf.indicatorType = .activity
        profileView.profileImageView.kf.setImage(with: imageURL,
                                    placeholder: UIImage(named: "stub"),
                                    options: [.processor(processor)])
    }
}
```

```swift
import UIKit

final class ProfileView: UIView {
    
    let profileImageView: UIImageView = {
        $0.translatesAutoresizingMaskIntoConstraints = false
        $0.contentMode = .scaleAspectFill
        $0.layer.cornerRadius = $0.frame.size.width / 2
        $0.clipsToBounds = true
        return $0
    }(UIImageView(image: UIImage(named: "user")))
    
    var profileName: UILabel? = {
        $0.translatesAutoresizingMaskIntoConstraints = false
        $0.text = "Екатерина Новикова"
        $0.font = UIFont.boldSystemFont(ofSize: 23)
        $0.textColor = UIColor(red: 255/255, green: 255/255, blue: 255/255, alpha: 1)
        $0.numberOfLines = 1
        return $0
    }(UILabel(frame: .zero))
    
    var profileLogin: UILabel? = {
        $0.translatesAutoresizingMaskIntoConstraints = false
        $0.text = "@ekaterina_nov"
        $0.font = .preferredFont(forTextStyle: .footnote)
        $0.textColor = UIColor(red: 174/255, green: 175/255, blue: 180/255, alpha: 1)
        return $0
    }(UILabel(frame: .zero))
    
    var profileDescription: UILabel? = {
        $0.translatesAutoresizingMaskIntoConstraints = false
        $0.text = "Hello, world!"
        $0.font = .preferredFont(forTextStyle: .footnote)
        $0.textColor = UIColor(red: 255/255, green: 255/255, blue: 255/255, alpha: 1)
        return $0
    }(UILabel(frame: .zero))
    
    private let logOutButton: UIButton = {
        $0.translatesAutoresizingMaskIntoConstraints = false
        $0.tintColor = UIColor.ypRed
        let image = UIImage(named: "ipad.and.arrow.forward")
        $0.setImage(image, for: .normal)
        $0.addTarget(self, action: #selector(didTapLogoutButton), for: .touchUpInside)
      return $0
    }(UIButton(type: .custom))
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = UIColor(red: 26/255, green: 27/255, blue: 34/255, alpha: 1)
        setupContraints()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    @objc
    private func didTapLogoutButton() {
        
    }
}

extension ProfileView {
    private func setupContraints() {
        addSubview(profileImageView)
        addSubview(profileName!)
        addSubview(profileLogin!)
        addSubview(profileDescription!)
        addSubview(logOutButton)
        
        // Провести проверку по корректности лейблов (profileLogin, profileDescription) - trailing нужен ли для них или нет.
        NSLayoutConstraint.activate([
            profileImageView.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 32),
            profileImageView.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            profileImageView.heightAnchor.constraint(equalToConstant: 70),
            profileImageView.widthAnchor.constraint(equalToConstant: 70),
            
            profileName!.topAnchor.constraint(equalTo: profileImageView.bottomAnchor, constant: 8),
            profileName!.leadingAnchor.constraint(equalTo: profileImageView.leadingAnchor),
             // Чтобы был отступ справа если будет к примеру такое имя: Константин Константинопольский
            profileName!.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            
            profileLogin!.topAnchor.constraint(equalTo: profileName!.bottomAnchor, constant: 8),
            profileLogin!.leadingAnchor.constraint(equalTo: profileName!.leadingAnchor),
            profileLogin!.trailingAnchor.constraint(equalTo: profileName!.trailingAnchor),
            
            profileDescription!.topAnchor.constraint(equalTo: profileLogin!.bottomAnchor, constant: 8),
            profileDescription!.leadingAnchor.constraint(equalTo: profileName!.leadingAnchor),
            profileDescription!.trailingAnchor.constraint(equalTo: profileLogin!.trailingAnchor),
            
            logOutButton.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -20),
            logOutButton.centerYAnchor.constraint(equalTo: profileImageView.centerYAnchor)
        ])
    }
}
```


